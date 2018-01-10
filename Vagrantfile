# -*- mode: ruby -*-
# vi: set ft=ruby :

load 'networkconfig/default'

Dir.mkdir("./Sites") unless File.exists?("./Sites")
Vagrant.configure("2") do |config|

    config.vm.box_version = "2.5"
    config.vm.box = "scotch/box"
    config.vm.network "#{NETWORK_TYPE}", ip: "#{IP}", bridge: "en0: Wi-Fi (AirPort)"
    config.vm.hostname = "scotchbox"
    config.vm.synced_folder "./Sites", "/var/www", :mount_options => ["dmode=777", "fmode=666"]
    config.vm.synced_folder "./Databases", "/var/lib/mysql", :mount_options => ["dmode=700", "fmode=660"], owner: "mysql", group: "mysql"

    config.vm.provision "shell", run: "always", inline: <<-SHELL

        while IFS= read -r line; do
            if [ "$line" != "" ]; then
                DOMAINS+=("$line")
            fi
        done < <(grep "" /vagrant/Domains)

        ## Disable all sites
        cd /etc/apache2/sites-available/
        a2dissite *.conf > /dev/null 2>&1
        rm *.conf
        
        ## Loop through all sites
        for ((i=0; i < ${#DOMAINS[@]}; i++)); do

            ## Current Domain
            DOMAIN=${DOMAINS[$i]}

            mkdir -p /var/www/$DOMAIN/logs
            mkdir -p /var/www/$DOMAIN/public

            sudo cp /vagrant/site.template /etc/apache2/sites-available/$DOMAIN.conf
            sudo sed -i s,site.template,$DOMAIN,g /etc/apache2/sites-available/$DOMAIN.conf
            sudo a2ensite $DOMAIN.conf > /dev/null 2>&1

        done

        sudo service apache2 restart > /dev/null 2>&1
        sudo service mysql restart> /dev/null 2>&1

    SHELL

    # Mailcatcher
    config.vm.provision "shell", inline: "/home/vagrant/.rbenv/shims/mailcatcher --http-ip=0.0.0.0", run: "always"

end
