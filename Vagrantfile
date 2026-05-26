Vagrant.configure("2") do |config|
  config.vm.box = "debian/trixie64"
  config.vm.box_version = "13.20260519.1"

  mem = (ENV['VAGRANT_MEM'] || 4096).to_i
  cpus = (ENV['VAGRANT_CPUS'] || 4).to_i
  gsd_project_dir = ENV['GSD_PROJECT_DIR'] || nil
  # determine when to require the env var
  actionable_commands = %w[up reload resume]
  invoked = ARGV.find { |a| actionable_commands.include?(a) }
  if invoked
    raise "Environment variable GSD_PROJECT_DIR must be set (e.g. export GSD_PROJECT_DIR=/path)" unless gsd_project_dir
    raise "GSD_PROJECT_DIR points to missing directory: #{gsd_project_dir}" unless File.directory?(gsd_project_dir)
  end

  config.vm.provider :libvirt do |lv|
    lv.memory = mem
    lv.cpus   = cpus
  end
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "~/.gsd", "/home/vagrant/.gsd", type: "nfs", nfs_version: 4
  if invoked
    config.vm.synced_folder gsd_project_dir, "/home/vagrant/" + File.basename(gsd_project_dir), type: "nfs", nfs_version: 4
  end

  config.vm.provision "shell", inline: <<-SHELL
    set -e

    apt-get update -y
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
      git \
      curl \
      direnv \
      xauth \
      x11-apps \
      dbus-user-session \
      fonts-dejavu-core \
      bash-completion

    DIRENV_SHELLHOOK='eval "$(direnv hook bash)"'
    BASHRC_PATH="/home/vagrant/.bashrc"
    grep -qxF "$DIRENV_SHELLHOOK" "$BASHRC_PATH" || echo "$DIRENV_SHELLHOOK" >> "$BASHRC_PATH"

    curl -fsSL https://deb.nodesource.com/setup_24.x | bash -
    apt-get install -y nodejs
    npm install -g @opengsd/gsd-pi@1.0.2

    # ensure SSH allows X11Forwarding (usually default)
    sed -i 's/^#X11Forwarding yes/X11Forwarding yes/' /etc/ssh/sshd_config || true
    sed -i 's/^#X11UseLocalhost yes/X11UseLocalhost yes/' /etc/ssh/sshd_config || true
    systemctl restart ssh || service ssh restart || true

    test -e "/nix" || \
      curl --proto '=https' --tlsv1.2 -sSf -L https://install.lix.systems/lix | \
      sh -s -- install linux --no-confirm
  SHELL
end
