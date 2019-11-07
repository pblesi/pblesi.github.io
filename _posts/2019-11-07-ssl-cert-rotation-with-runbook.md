---
layout: post
title: SSL Cert Rotation with Runbook
image: /assets/img/ssl_cert_rotation_with_runbook_01.png
date: '2019-11-07T11:00:00.000-05:00'
author: Patrick Blesi
comments: true
tags:
  - Automation
  - Devops
  - Ruby
  - Runbook
modified_time: '2019-11-07T11:00:00.000-05:00'
---

<p>
  <img class="img" src="/assets/img/ssl_cert_rotation_with_runbook_01.png" width="750">
</p>

SSL cert rotation is a problem that plagues nearly every web developer. Some are fortunate to work with infrastructure where an automated solution is available. This eliminates the need for tedious SSL cert management. Others are fortunate to have a team dedicated to managing infrastructure needs, so they can toss this issue to another engineer.

Unfortunately, I do not fall into either of these two camps when it comes to SSL certificate management. I needed to develop my own certificate management solution to ensure new SSL certs got properly installed on my servers. This post covers how to use Braintree's [Runbook](https://github.com/braintree/runbook) to automate the rotation of SSL certificates.

My certificate infrastructure is based on [Jamie Nguyen's OpenSSL Certificate Authority Tutorial](https://jamielinux.com/docs/openssl-certificate-authority/). I store a root certificate authority and intermediate certificate authority on my local machine. From the intermediate CA, I issue certs for servers within my infrastructure. All servers in my infrastructure trust my intermediate CA, so they accept traffic from valid certs when connecting to other servers.

SSL certificate rotation is a process that's ripe for automation. It is a tedious, repetitive process. The process is performed infrequently enough that it is easy to forget how to do it. And it is mission critical; failing to properly rotate SSL certificates almost certainly results in an outage.

## The automation process

Several iterations were required before I boiled the rotation process down to a single command entered on the command line. My first pass at rotating SSL certs involved a lot of trial and error. Ultimately, I nailed down all the steps required to successfully rotate SSL certificates in my infrastructure. I took diligent notes to be well-armed for my next inevitable encounter with SSL cert rotation.

On my second iteration, with notes in hand, I was able to  successfully rotate certificates with only minor tweaks to my documented process. Nevertheless, having a handful of servers still made this a tedious process. Retyping the several-dozen commmand incantations over and over for each server is not how I like to spend my afternoons. Leveraging [Runbook](https://github.com/braintree/runbook) allowed me to bring this process down to single command for rotating my SSL certs. Below is the [SSL cert rotation runbook](https://gist.github.com/pblesi/a48e2a2c07cd22f3e0cab49d3888e724) I developed.

## The runbook

The SSL Cert rotation process breaks down into three main steps. First, a new cert needs to be generated using the intermediate CA. Second, the new cert needs to be distributed to the target server. And finally, the service must be restarted to pick up the new certificate.

See [Runbook's README.md](https://github.com/braintree/runbook) for details on setting up and working with Runbook.

### Setup

First, we must collect some input for our parameterized runbook. The two pieces of information we need are the host and service for the SSL certificate we are generating.

```ruby
#!/usr/bin/env ruby
require "runbook"

host = ENV["HOST"] # e.x. ldap01.stg
raise "Error no host specified using HOST env var" unless host

service = ENV["SERVICE"] # e.x. slapd
raise "Error no service specified using SERVICE env var" unless service

local_user = ENV["USER"]
company_name = "patricks_pickles"
intermediate_ca_path = "/root/ca/intermediate"
local_git_dir = "/home/#{local_user}/dev/#{company_name}/pp-infrastructure"
local_cert_path = "modules/ldap/files"
local_cert_file = "#{host}_#{service}.cert.pem"
staging_suffix = "#{company_name}-staging.com"
prod_suffix = "#{company_name}.com"
```

We collect this info from the command line using environment variables. The reason we do not use [Runbook's ask statement](https://github.com/braintree/runbook#ask) to collect this info is because these values are used prominently throughout the runbook. It makes for a cleaner implementation to collect these values on the command line and then parameterize our runbook with them before executing it.

Additionally, we initialize a number of other local variables that are used throughout our runbook.

All runbooks are initialized with a title. It is best-practice to provide a description of the runbook as well.

```ruby
runbook = Runbook.book "Renew SSL Certs" do
  description <<-DESC
    This Runbook rotates SSL Certs.
  DESC

  layout [[:runbook, :commands]]

  section "Setup" do
    step { ruby_command { @env = host.split(".").last.to_sym } }
  end
end
```

This runbook has a layout, which implies the runbook manages multiple panes using [tmux](https://github.com/tmux/tmux). The layout is a stacked layout where the runbook executes in the top pane and commands are sent to the bottom pane. The setup section sets the `@env` instance variable at runtime. This value is subsequently available in all other `ruby_command` blocks.

### Certificate generation

After initial setup, the next runbook section encompasses all steps required to create the new certificate.

```ruby
runbook = Runbook.book "Renew SSL Certs" do
  #...

  section "Create New Cert" do
    user "root"

    step "Backup and update index.txt" do
      capture %Q{ls #{intermediate_ca_path}/index.txt.old* | tail -n 1 | sed -E "s/.*([0-9]{2})/\\1/"}, into: :backup_num

      ruby_command do
        @backup_num = (backup_num.to_i + 1).to_s.rjust(2, "0")

        command "cp #{intermediate_ca_path}/index.txt #{intermediate_ca_path}/index.txt.old#{@backup_num}"
        command "cp #{intermediate_ca_path}/index.txt.attr #{intermediate_ca_path}/index.txt.attr.old#{@backup_num}"

        command %Q{sed -i "/#{host} #{service.upcase}/d" #{intermediate_ca_path}/index.txt}
      end
    end

    step "Generate new cert" do
      ruby_command do
        case env
        when :stg
          @expiration_days = 1035
        when :prod
          @expiration_days = 1095
        else
          raise "Unknown env: #{env}"
        end

        tmux_command "sudo openssl ca -config #{intermediate_ca_path}/openssl.cnf -extensions server_cert -days #{@expiration_days} -notext -md sha256 -in #{intermediate_ca_path}/csr/#{host}_#{service}.csr.pem -out #{intermediate_ca_path}/certs/#{local_cert_file}", :commands
        confirm "Have you generated the cert?"
      end

      command "sudo chmod 444 #{intermediate_ca_path}/certs/#{local_cert_file}"

      tmux_command "sudo openssl x509 -noout -text -in #{intermediate_ca_path}/certs/#{local_cert_file}", :commands
      confirm "Does the cert look correct?"

      tmux_command "sudo openssl verify -CAfile #{intermediate_ca_path}/certs/ca-chain.cert.pem #{intermediate_ca_path}/certs/#{local_cert_file}", :commands
      confirm "Is the cert valid?"
    end
  end
end
```

The `user` setter designates that all commands in this section exececute as the `root` user. Because no host is specified, commands are executed locally. `confirm` statements allow us to confirm everything is correct before moving on to the next step.

Because the case statement references the `env` instance variable, it must be wrapped in a `ruby_command` block so it is executed at runtime. Because the `tmux_command` references `@expiration_days`, it also must be defined in the `ruby_command` block.

### Uploading the certificate

In this section we copy the newly-generated certificate to our version-controlled infrastructure management repository, upload the certificate to the server, and install the certificate.

```ruby
runbook = Runbook.book "Renew SSL Certs" do
  #...

  section "Upload Cert" do
    step "Copy the cert to pp-infrastructure" do
      user "root"

      command "cp #{intermediate_ca_path}/certs/#{local_cert_file} #{local_git_dir}/#{local_cert_path}"
      command "chown #{local_user}:#{local_user} #{local_git_dir}/#{local_cert_path}/#{local_cert_file}"
    end

    step "Upload the cert" do
      server host

      upload "#{local_git_dir}/#{local_cert_path}/#{local_cert_file}", to: "/home/#{local_user}"
    end

    step "Install the cert" do
      server host
      user "root"

      command "mv ~#{local_user}/#{local_cert_file} /etc/ssl"
      command "chown root:ssl-cert /etc/ssl/#{local_cert_file}"
      command "chmod 444 /etc/ssl/#{local_cert_file}"

      command "cp /etc/ssl/#{local_cert_file} /etc/ssl/certs"
    end
  end
end
```

The `server` setter uses entries in my `~/.ssh/config` file to resolve the long-form host name, port, and user to log in as for a given host string passed to ssh.

### Restarting the service

Lastly, we restart the service, verify our new certificate is valid, and commit our certificate to version control.

```ruby
runbook = Runbook.book "Renew SSL Certs" do
  #...

  section "Upload Cert" do
    #...

    step "Restart the service" do
      server host
      user "root"

      command "service #{service} restart"
    end

    step "Validate the service cert is valid" do
      ruby_command do
        case env
        when :stg
          @suffix = staging_suffix
        when :prod
          @suffix = prod_suffix
        else
          raise "Unknown env: #{env}"
        end

        tmux_command "ssh #{host}", :commands
        tmux_command "openssl s_client -connect #{host}.#{@suffix}:12345 -CApath /etc/ssl/certs", :commands

        confirm "Is the cert valid?"
      end
    end

    step "Commit cert changes" do
      path "#{local_git_dir}"

      command "git add #{local_cert_path}/#{local_cert_file}"
      command %Q{git commit -m "Update #{local_cert_file} certificate"}
      command "git push"
    end
  end
end
```

Including the cert validation step allows us to confirm that the new cert is working as expected so we don't encounter an issue when the old certificate expires.

With our runbook in hand, rotating SSL certs is as simple as invoking the following command:

```bash
HOST=ldap01.stg SERVICE=slapd runbook exec runbooks/renew_ssl_certs.rb
```

## Conclusion

The full runbook for rotating SSL certs is available as a [gist](https://gist.github.com/pblesi/a48e2a2c07cd22f3e0cab49d3888e724). This runbook likely won't meet your SSL rotation needs as is. But hopefully it can serve as a basis for automating your SSL cert rotation process, so you are no longer plagued with manually managing SSL certificates.
