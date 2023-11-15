Vagrant.configure("2") do |config|
  config.vm.define "fbsd_12" do |c|
    c.vm.box = "roboxes/freebsd12"
  end

  config.vm.define "fbsd_13" do |c|
    c.vm.box = "roboxes/freebsd13"
  end

  config.vm.define "fbsd_14" do |c|
    c.vm.box = "roboxes/freebsd14"
  end

  config.vm.provider "libvirt" do |qe|
    # https://vagrant-libvirt.github.io/vagrant-libvirt/configuration.html
    qe.driver = "kvm"
    qe.cpus = 1
    qe.memory = 8192
  end

  config.vm.boot_timeout = 600

  config.vm.provision "shell", inline: <<~SHELL
    set -e

    pkg bootstrap
    pkg update
    pkg install -y curl bash git gmake sudo llvm libyaml
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
    rbenv install 3.2.2 || (cat /tmp/ruby-build*.log && exit 1)
    rbenv rehash
    rbenv global 3.2.2
    # Clean up any lingering documentation files
    shopt -s globstar
    rm -rf ~/.rbenv/versions/**/ri

    curl https://sh.rustup.rs -sSf | sh -s -- -y --profile minimal --default-toolchain 1.72.1
    EOF

    echo 'Disk usage before cleanup:'
    df -h
    du -hs /home/vagrant
    du -ah / | sort -r -h | head -25

    echo 'Cleaning up packages'
    pkg prime-list
    pkg clean

    echo 'Cleaning up unneeded directories'
    rm -rf /usr/obj /usr/ports /usr/src /usr/tests /usr/lib/debug /var/db

    # Adapted from bento's minimize.sh. This writes zeroes to the disk so that the empty space is
    # easily compressed.
    echo 'Zeroing out empty space'
    dd if=/dev/zero of=/EMPTY bs=1G count=4 || true
    rm -f /EMPTY
    sync

    echo 'Disk usage after cleanup:'
    df -h
    du -hs /home/vagrant
    du -ah / | sort -r -h | head -25
  SHELL
end
