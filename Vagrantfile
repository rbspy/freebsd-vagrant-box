Vagrant.configure("2") do |config|
  config.vm.define "fbsd_13_0" do |fbsd_13_0|
    fbsd_13_0.vm.box = "freebsd/FreeBSD-13.0-RELEASE"
  end

  config.vm.define "fbsd_12_2" do |fbsd_12_2|
    fbsd_12_2.vm.box = "freebsd/FreeBSD-12.2-STABLE"
  end

  config.vm.synced_folder ".", "/vagrant", type: "rsync",
    rsync__exclude: ".git/",
    rsync__auto: true

  config.vm.provision "shell", inline: <<~SHELL
    pkg bootstrap
    pkg install -y curl bash git gmake llvm

    su vagrant <<EOF
    git clone https://github.com/rbenv/rbenv.git ~/.rbenv
    git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
    echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bash_profile
    echo '~/.rbenv/bin/rbenv init' >> ~/.bash_profile

    env MAKEFLAGS="-j2" ~/.rbenv/bin/rbenv install 2.7.4
    env MAKEFLAGS="-j2" ~/.rbenv/bin/rbenv install 3.0.2
    ~/.rbenv/bin/rbenv global 3.0.2

    curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain 1.56.0
    EOF
  SHELL

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  config.vm.boot_timeout = 600
end
