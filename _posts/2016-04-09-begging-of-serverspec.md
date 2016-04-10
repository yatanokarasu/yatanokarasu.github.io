---
layout:         post
title:          "The beginning os Serverspec"
keywords:       [Serverspec]
categories:     [Maintainance, Operation]
tags:           [Serverspec]
---

[Serverspec](http://serverspec.org/) provides functions to test server status like pakcages, existence of files, etc.

<!-- more -->

# Installing

If you use Msys2 on Windows OS and already installed ruby, you can install Serverspec using by the following command:

~~~
> gem install serverspec
~~~


# Getting started

You have only to know two commands as follows:

1.  `serverspec-init`: Create Serverspec workspace
2.  `rake spec`: Run serverspec tests


Run `serverspec-init` command, and answer some question interactively.

<div class="notice">Run `serverspsec-init` on `mintty.exe` on Windows OS, Servers spec prompt will not be displayed, but running just invisible. So you can do user interaction.</div>

~~~
$ serverspec-init
Select OS type:Select OS type:

  1) UN*X
  2) Windows

Select number: 1                <= ** Test target is UNIX type OS **
Select a backend type:

  1) SSH
  2) Exec (local)

Select number: 2                        <= ** Run test via SSH connection **
Vagrant instance y/n: n                 <= ** Test target is not Vagrant environment **
Input target host name: 192.168.0.254   <= ** Target host name is '192.168.0.254' **
 + spec/
 + spec/192.168.0.254/
 + spec/192.168.0.254/sample_spec.rb
 + spec/spec_helper.rb
 + Rakefile
 + .rspec

$
~~~


The `sample_spec.rb` file create as default tests whether `httpd` pakcages is installed and running or not.

<span class="balloon">spec/test/sample_spec.rc</span>

~~~
require 'spec_helper'

describe package('httpd'), :if => os[:family] == 'redhat' do
  it { should be_installed }
end

describe package('apache2'), :if => os[:family] == 'ubuntu' do
  it { should be_installed }
end

describe service('httpd'), :if => os[:family] == 'redhat' do
  it { should be_enabled }
  it { should be_running }
end

describe service('apache2'), :if => os[:family] == 'ubuntu' do
  it { should be_enabled }
  it { should be_running }
end

describe service('org.apache.httpd'), :if => os[:family] == 'darwin' do
  it { should be_enabled }
  it { should be_running }
end

describe port(80) do
  it { should be_listening }
end
~~~


The following command run tests:

~~~
$ rake spec

Package "httpd"
  should be installed (FAILED - 1)

Service "httpd"
  should be enabled (FAILED - 2)
  should be running (FAILED - 3)

Port "80"
  should be listening (FAILED - 4)

Package "httpd"
  should be installed (FAILED - 5)

Service "httpd"
  should be enabled (FAILED - 6)
  should be running (FAILED - 7)

Port "80"
  should be listening (FAILED - 8)

Failures:

  1) Package "httpd" should be installed
     On host `192.168.0.254'
     Failure/Error: it { should be_installed }
       expected Package "httpd" to be installed
       sudo -p 'Password: ' /bin/sh -c rpm\ -q\ httpd
       package httpd is not installed

     # ./spec/192.168.0.254/sample_spec.rb:4:in `block (2 levels) in <top (required)>'

  2) Service "httpd" should be enabled
     On host `192.168.0.254'
     Failure/Error: it { should be_enabled }
       expected Service "httpd" to be enabled
       sudo -p 'Password: ' /bin/sh -c chkconfig\ --list\ httpd\ \|\ grep\ 3:on

     # ./spec/192.168.0.254/sample_spec.rb:12:in `block (2 levels) in <top (required)>'

  3) Service "httpd" should be running
     On host `192.168.0.254'
     Failure/Error: it { should be_running }
       expected Service "httpd" to be running
       sudo -p 'Password: ' /bin/sh -c ps\ aux\ \|\ grep\ -w\ --\ httpd\ \|\ grep\ -qv\ grep

     # ./spec/192.168.0.254/sample_spec.rb:13:in `block (2 levels) in <top (required)>'

  4) Port "80" should be listening
     On host `192.168.0.254'
     Failure/Error: it { should be_listening }
       expected Port "80" to be listening
       sudo -p 'Password: ' /bin/sh -c netstat\ -tunl\ \|\ grep\ --\ :80\\\

     # ./spec/192.168.0.254/sample_spec.rb:27:in `block (2 levels) in <top (required)>'

  5) Package "httpd" should be installed
     On host `192.168.0.254'
     Failure/Error: it { should be_installed }
       expected Package "httpd" to be installed
       sudo -p 'Password: ' /bin/sh -c rpm\ -q\ httpd
       package httpd is not installed

     # ./spec/192.168.0.254/sample_spec.rb:4:in `block (2 levels) in <top (required)>'

  6) Service "httpd" should be enabled
     On host `192.168.0.254'
     Failure/Error: it { should be_enabled }
       expected Service "httpd" to be enabled
       sudo -p 'Password: ' /bin/sh -c chkconfig\ --list\ httpd\ \|\ grep\ 3:on

     # ./spec/192.168.0.254/sample_spec.rb:12:in `block (2 levels) in <top (required)>'

  7) Service "httpd" should be running
     On host `192.168.0.254'
     Failure/Error: it { should be_running }
       expected Service "httpd" to be running
       sudo -p 'Password: ' /bin/sh -c ps\ aux\ \|\ grep\ -w\ --\ httpd\ \|\ grep\ -qv\ grep

     # ./spec/192.168.0.254/sample_spec.rb:13:in `block (2 levels) in <top (required)>'

  8) Port "80" should be listening
     On host `192.168.0.254'
     Failure/Error: it { should be_listening }
       expected Port "80" to be listening
       sudo -p 'Password: ' /bin/sh -c netstat\ -tunl\ \|\ grep\ --\ :80\\\

     # ./spec/192.168.0.254/sample_spec.rb:27:in `block (2 levels) in <top (required)>'

Finished in 2.27 seconds (files took 14.59 seconds to load)
8 examples, 8 failures

Failed examples:

rspec ./spec/192.168.0.254/sample_spec.rb:4 # Package "httpd" should be installed
rspec ./spec/192.168.0.254/sample_spec.rb:12 # Service "httpd" should be enabled
rspec ./spec/192.168.0.254/sample_spec.rb:13 # Service "httpd" should be running
rspec ./spec/192.168.0.254/sample_spec.rb:27 # Port "80" should be listening
rspec ./spec/192.168.0.254/sample_spec.rb:4 # Package "httpd" should be installed
rspec ./spec/192.168.0.254/sample_spec.rb:12 # Service "httpd" should be enabled
rspec ./spec/192.168.0.254/sample_spec.rb:13 # Service "httpd" should be running
rspec ./spec/192.168.0.254/sample_spec.rb:27 # Port "80" should be listening

ruby -I'/lib/ruby/gems/2.3.0/gems/rspec-support-3.4.1/lib';'/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/lib' '/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/exe/rspec' --pattern 'spec/192.168.0.254/*_spec.rb' failed
ruby -I'/lib/ruby/gems/2.3.0/gems/rspec-support-3.4.1/lib';'/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/lib' '/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/exe/rspec' --pattern 'spec/192.168.0.254/*_spec.rb'

~~~


After installing 'httpd' on '192.168.0.254', you can see that a test case is passed, which checks httpd is installed.

~~~
# yum -y install httpd

------------------------------

$ rake spec

Package "httpd"
  should be installed

Service "httpd"
  should be enabled (FAILED - 1)
  should be running (FAILED - 2)

Port "80"
  should be listening (FAILED - 3)

Package "httpd"
  should be installed

Service "httpd"
  should be enabled (FAILED - 4)
  should be running (FAILED - 5)

Port "80"
  should be listening (FAILED - 6)

Failures:

  1) Service "httpd" should be enabled
     On host `192.168.0.254'
     Failure/Error: it { should be_enabled }
       expected Service "httpd" to be enabled
       sudo -p 'Password: ' /bin/sh -c chkconfig\ --list\ httpd\ \|\ grep\ 3:on

     # ./spec/192.168.0.254/sample_spec.rb:12:in `block (2 levels) in <top (required)>'

  2) Service "httpd" should be running
     On host `192.168.0.254'
     Failure/Error: it { should be_running }
       expected Service "httpd" to be running
       sudo -p 'Password: ' /bin/sh -c service\ httpd\ status
       httpd is stopped

     # ./spec/192.168.0.254/sample_spec.rb:13:in `block (2 levels) in <top (required)>'

  3) Port "80" should be listening
     On host `192.168.0.254'
     Failure/Error: it { should be_listening }
       expected Port "80" to be listening
       sudo -p 'Password: ' /bin/sh -c netstat\ -tunl\ \|\ grep\ --\ :80\\\

     # ./spec/192.168.0.254/sample_spec.rb:27:in `block (2 levels) in <top (required)>'

  4) Service "httpd" should be enabled
     On host `192.168.0.254'
     Failure/Error: it { should be_enabled }
       expected Service "httpd" to be enabled
       sudo -p 'Password: ' /bin/sh -c chkconfig\ --list\ httpd\ \|\ grep\ 3:on

     # ./spec/192.168.0.254/sample_spec.rb:12:in `block (2 levels) in <top (required)>'

  5) Service "httpd" should be running
     On host `192.168.0.254'
     Failure/Error: it { should be_running }
       expected Service "httpd" to be running
       sudo -p 'Password: ' /bin/sh -c service\ httpd\ status
       httpd is stopped

     # ./spec/192.168.0.254/sample_spec.rb:13:in `block (2 levels) in <top (required)>'

  6) Port "80" should be listening
     On host `192.168.0.254'
     Failure/Error: it { should be_listening }
       expected Port "80" to be listening
       sudo -p 'Password: ' /bin/sh -c netstat\ -tunl\ \|\ grep\ --\ :80\\\

     # ./spec/192.168.0.254/sample_spec.rb:27:in `block (2 levels) in <top (required)>'

Finished in 1.98 seconds (files took 13.86 seconds to load)
8 examples, 6 failures

Failed examples:

rspec ./spec/192.168.0.254/sample_spec.rb:12 # Service "httpd" should be enabled
rspec ./spec/192.168.0.254/sample_spec.rb:13 # Service "httpd" should be running
rspec ./spec/192.168.0.254/sample_spec.rb:27 # Port "80" should be listening
rspec ./spec/192.168.0.254/sample_spec.rb:12 # Service "httpd" should be enabled
rspec ./spec/192.168.0.254/sample_spec.rb:13 # Service "httpd" should be running
rspec ./spec/192.168.0.254/sample_spec.rb:27 # Port "80" should be listening

ruby -I'/lib/ruby/gems/2.3.0/gems/rspec-support-3.4.1/lib';'/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/lib' '/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/exe/rspec' --pattern 'spec/192.168.0.254/*_spec.rb' failed
ruby -I'/lib/ruby/gems/2.3.0/gems/rspec-support-3.4.1/lib';'/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/lib' '/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/exe/rspec' --pattern 'spec/192.168.0.254/*_spec.rb'

~~~


To pass this test successfully, enable and run 'httpd' service, and then rerun this test.


~~~
# chkconfig --level 3 httpd on
# service httpd start

------------------------------

$ rake spec

Package "httpd"
  should be installed

Service "httpd"
  should be enabled
  should be running

Port "80"
  should be listening

Package "httpd"
  should be installed

Service "httpd"
  should be enabled
  should be running

Port "80"
  should be listening

Finished in 1.64 seconds (files took 13.08 seconds to load)
8 examples, 0 failures

ruby -I'/lib/ruby/gems/2.3.0/gems/rspec-support-3.4.1/lib';'/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/lib' 'C:/msys64/mingw64/lib/ruby/gems/2.3.0/gems/rspec-core-3.4.4/exe/rspec' --pattern 'spec/192.168.0.254/*_spec.rb'

~~~

