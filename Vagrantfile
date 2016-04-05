Vagrant.configure(2) do |config|
  config.vm.box = "bento/ubuntu-14.04"
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.customize ["modifyvm", :id, "--memory", 4096]
    vb.customize ["modifyvm", :id, "--cpus", `#{RbConfig::CONFIG['host_os'] =~ /darwin/ ? 'sysctl -n hw.ncpu' : 'nproc'}`.chomp]
    vb.customize ["modifyvm", :id, "--usb", 'on']
    vb.customize ["usbfilter", "add", "0", 
                       "--target", :id,
                       "--name", "SDR",
                       "--manufacturer", "Realtek",
                       "--product", "RTL2838UHIDIR"]
  end

  ["vmware_fusion", "vmware_workstation"].each do |vmware|
        config.vm.provider vmware do |v|
          v.gui = false
          v.vmx['memsize'] = 4096
          v.vmx['numvcpus'] = 4
        end
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get install -y software-properties-common

    sudo bash -c "echo 'Etc/UTC' > /etc/timezone"
    sudo dpkg-reconfigure -f noninteractive tzdata

    sudo apt-mark hold grub
    sudo apt-mark hold grub2-common
    sudo apt-mark hold grub-gfxpayload-lists
    sudo apt-mark hold grub-pc
    sudo apt-mark hold grub-common

    sudo apt-get update
    sudo apt-get upgrade -y

    sudo apt-get install --no-install-recommends -y git debconf-utils libusb-1.0-0-dev \
        cmake sdcc qtcreator rar unrar libpcsclite-dev realpath \
        libc6-dev-i386 python-mysqldb tshark wireshark mplayer sox \
        ntp ntpdate autoconf zip unzip libtool libtevent-dev sqlite

    sudo apt-get install --no-install-recommends -y git-core cmake g++ python-dev swig \
        pkg-config libfftw3-dev libboost1.55-all-dev libcppunit-dev libgsl0-dev \
        libusb-dev libsdl1.2-dev python-wxgtk2.8 python-numpy \
        python-cheetah python-lxml doxygen libxi-dev python-sip \
        libqt4-opengl-dev libqwt-dev libfontconfig1-dev libxrender-dev \
        python-sip python-sip-dev python-gtk2 python-sphinx python-numpy python-cheetah \
	python-comedilib libcomedi-dev libzmq-dev automake

    sudo apt-get install --no-install-recommends -y libuhd-dev

    sudo debconf-set-selections <<< 'wireshark-common wireshark-common/install-setuid boolean true'
    sudo usermod -aG wireshark vagrant

    sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password tetra'
    sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password tetra'

    sudo apt-get install -y apache2 mysql-server
    sudo apt-get install -y php5
    sudo /etc/init.d/mysql start

    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/dbconfig-install boolean true'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/app-password-confirm password tetra'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/mysql/admin-pass password tetra'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/password-confirm password tetra'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/setup-password password tetra'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/database-type select mysql'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/mysql/app-pass password tetra'
    sudo debconf-set-selections <<< 'phpmyadmin phpmyadmin/reconfigure-webserver multiselect none'
		
    sudo debconf-set-selections <<< 'dbconfig-common dbconfig-common/mysql/app-pass password tetra'
    sudo debconf-set-selections <<< 'dbconfig-common dbconfig-common/password-confirm password tetra'
    sudo debconf-set-selections <<< 'dbconfig-common dbconfig-common/app-password-confirm password tetra'
    sudo debconf-set-selections <<< 'dbconfig-common dbconfig-common/app-password-confirm password tetra'
    sudo debconf-set-selections <<< 'dbconfig-common dbconfig-common/password-confirm password tetra'
        
    sudo apt-get -y install phpmyadmin

    sudo sed -i 's/max_execution_time = .*/max_execution_time = 90/g' /etc/php5/apache2/php.ini
    sudo sed -i 's/memory_limit = .*/memory_limit = 1024M/g' /etc/php5/apache2/php.ini
    sudo sed -i 's/post_max_size = .*/post_max_size = 1024M/g' /etc/php5/apache2/php.ini
    sudo sed -i 's/upload_max_filesize = .*/upload_max_filesize = 1024/g' /etc/php5/apache2/php.ini
    sudo service apache2 restart

    sudo apt-get autoremove

    sudo bash
    pwd
    id

    mkdir -p ~/install
    cd ~/install
    git clone git://git.osmocom.org/rtl-sdr.git
    mkdir -p rtl-sdr/build
    cd rtl-sdr/build
    cmake ../ -DINSTALL_UDEV_RULES=ON
    make -j
    sudo make install
    sudo ldconfig
    sudo /etc/init.d/udev restart

    cd ~/install
    export PYTHONPATH=/opt/qt/lib/python2.7/dist-package
    git clone --recursive http://git.gnuradio.org/git/gnuradio.git
    mkdir -p gnuradio/build
    cd gnuradio/build
    cmake -DENABLE_DOXYGEN=OFF ..
    make -j 3
    make test
    sudo make install

    cd ~/install
    git clone git://git.osmocom.org/libosmocore
    cd libosmocore
    autoreconf -i -f
    ./configure
    make -j
    sudo make install
    sudo ldconfig

    cd ~/install
    git clone git://git.osmocom.org/gr-osmosdr
    mkdir -p gr-osmosdr/build
    cd gr-osmosdr/build
    cmake ..
    make -j
    sudo make install
    sudo ldconfig

    cd ~/install
    git clone https://github.com/csete/gqrx.git gqrx.git
    mkdir -p gqrx.git/build
    cd gqrx.git/build
    qmake ..
    make -j 3
    sudo make install

    sudo rmmod dvb_usb_rtl28xxu
    sudo bash -c "echo blacklist dvb_usb_rtl28xxu > /etc/modprobe.d/rtlsdr.conf"

    echo "CREATE DATABASE IF NOT EXISTS tetra;" | mysql -uroot -ptetra
    echo "CREATE USER IF NOT EXISTS 'tetra'@'localhost' IDENTIFIED BY 'tetra';" | mysql -uroot -ptetra
    echo "GRANT ALL PRIVILEGES ON tetra.* TO 'tetra'@'localhost' IDENTIFIED BY 'tetra' WITH GRANT OPTION;" | mysql -uroot -ptetra
    echo "FLUSH PRIVILEGES" | mysql -uroot -ptetra

    cd ~/
    git clone https://github.com/itds-consulting/tetra-listener.git --recursive
    cd tetra-listener
    ./build.sh

    chown -R vagrant:vagrant /home/vagrant

    shutdown -r now
    
  SHELL
end
