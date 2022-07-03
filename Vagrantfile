Vagrant.configure("2") do |config|
  config.vm.define "fbsd_13_0" do |fbsd_13_0|
    fbsd_13_0.vm.box = "freebsd/FreeBSD-13.0-RELEASE"
    config.disksize.size = "16GB"
  end

  config.vm.define "fbsd_12_2" do |fbsd_12_2|
    fbsd_12_2.vm.box = "freebsd/FreeBSD-12.2-STABLE"
    config.disksize.size = "16GB"
  end

  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provision "shell", inline: <<~SHELL
    pkg bootstrap
    pkg update
    pkg install -y curl bash git gmake sudo
    # Enable Vagrant's Synced Folders feature (sometimes flaky for FreeBSD guests)
    #pkg install -y virtualbox-ose-additions-nox11
    #sysrc vboxguest_enable="YES"
    #sysrc vboxservice_enable="YES"
    pkg clean -ay
    rm -rf /usr/ports /usr/share/doc

    chsh -s /usr/local/bin/bash vagrant
    pw groupmod wheel -m vagrant

    # Install ruby and rust toolchains
    su -l vagrant <<'EOF'
    git clone https://github.com/rbenv/rbenv.git ~/.rbenv
    git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bash_profile
    echo 'eval "$(rbenv init - bash)"' >> ~/.bash_profile

    source ~/.bash_profile

    MAKEFLAGS="-j2" rbenv install 2.7.6
    MAKEFLAGS="-j2" rbenv install 3.1.2
    rbenv rehash
    rbenv global 3.1.2

    curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain 1.62.0
    EOF

    pkg prime-list

    df -h
    du -hs /home/vagrant
  SHELL

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  config.vm.boot_timeout = 600
end
