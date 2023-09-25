Vagrant.configure("2") do |config|
  config.disksize.size = "16GB"

  config.vm.define "fbsd_13_2" do |c|
    c.vm.box = "freebsd/FreeBSD-13.2-RELEASE"
  end

  config.vm.define "fbsd_12_4" do |c|
    c.vm.box = "freebsd/FreeBSD-12.4-STABLE"
  end

  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provision "shell", inline: <<~SHELL
    pkg bootstrap
    pkg update
    pkg install -y curl bash git gmake sudo llvm
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

    export MAKEFLAGS="-j2"
    export RUBY_CONFIGURE_OPTS="--disable-install-doc"
    rbenv install 3.1.2
    rbenv rehash
    rbenv global 3.1.2
    # Clean up any lingering documentation files
    shopt -s globstar
    rm -rf ~/.rbenv/versions/**/ri

    curl https://sh.rustup.rs -sSf | sh -s -- -y --profile minimal --default-toolchain 1.72.1
    EOF

    pkg prime-list
    pkg clean

    rm -rf /usr/obj /usr/ports /usr/src /usr/tests /usr/lib/debug /var/db

    # Adapted from bento's minimize.sh. This writes zeroes to the disk so that the empty space is
    # easily compressed.
    dd if=/dev/zero of=/EMPTY bs=1M
    rm -f /EMPTY
    sync

    df -h
    du -hs /home/vagrant
    du -ah / | sort -r -h | head -25
  SHELL

  config.vm.provider "virtualbox" do |v|
    v.memory = 8192
    v.cpus = 3
  end

  config.vm.boot_timeout = 600
end
