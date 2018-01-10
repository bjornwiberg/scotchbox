# -*- mode: ruby -*-
# vi: set ft=ruby :

load 'networkconfig/default'

Dir.mkdir("./Sites") unless File.exists?("./Sites")
Dir.mkdir("./ssl") unless File.exists?("./ssl")

Vagrant.configure("2") do |config|

    config.vm.box_version = "2.5"
    config.vm.box = "scotch/box"
    config.vm.network "#{NETWORK_TYPE}", ip: "#{IP}", bridge: "en0: Wi-Fi (AirPort)"
    config.vm.hostname = "scotchbox"
    config.vm.synced_folder "./Sites", "/var/www", :mount_options => ["dmode=777", "fmode=666"]
    config.vm.synced_folder "./Databases", "/var/lib/mysql", :mount_options => ["dmode=700", "fmode=660"], owner: "mysql", group: "mysql"

    config.vm.provision "shell", run: "always", inline: <<-SHELL

        if ! [ -f /vagrant/Domains ]; then
            touch /vagrant/Domains
        fi

        while IFS= read -r line; do
            if [ "$line" != "" ]; then
                DOMAINS+=("$line")
            fi
        done < <(grep "" /vagrant/Domains)

        ## Enable ssl mod for apache
        a2enmod ssl > /dev/null 2>&1

        ## Disable all sites
        cd /etc/apache2/sites-available/
        a2dissite *.conf > /dev/null 2>&1
        rm *.conf > /dev/null 2>&1

        source /vagrant/.env

        if ! [ -f /vagrant/ssl/serial ]; then
            echo "1000" > /vagrant/ssl/serial
        fi

        if ! [ -f /vagrant/ssl/index.txt ]; then
            touch /vagrant/ssl/index.txt
        fi

        if ! [ -f /vagrant/ssl/private/ca.key.pem ]; then
            cd /vagrant/ssl && mkdir certs newcerts private

            # create root key
            openssl genrsa -out \
                /vagrant/ssl/private/ca.key.pem 4096 \
                > /dev/null 2>&1

            # create root cert
            openssl req -x509 \
                -config /vagrant/openssl.conf \
                -new \
                -nodes \
                -key /vagrant/ssl/private/ca.key.pem \
                -sha256 \
                -days 7300 \
                -subj "/C=$CA_COUNTRY/ST=$CA_STATE/L=$CA_CITY/O=$CA_ORGANISATION/CN=$CA_COMMONNAME" \
                -out /vagrant/ssl/certs/ca.cert.pem \
                > /dev/null 2>&1
        fi

        openssl=$(cat /vagrant/openssl.conf)
        
        ## Loop through all sites
        for ((i=0; i < ${#DOMAINS[@]}; i++)); do

            ## Current Domain
            DOMAIN=${DOMAINS[$i]}

            mkdir -p /var/www/$DOMAIN/logs > /dev/null 2>&1
            mkdir -p /var/www/$DOMAIN/public > /dev/null 2>&1
            mkdir -p /var/www/$DOMAIN/ssl/private/ > /dev/null 2>&1
            mkdir -p /var/www/$DOMAIN/ssl/certs/ > /dev/null 2>&1

            ## Check if there is a cert on server
            if ! [ -f /var/www/$DOMAIN/ssl/private/$DOMAIN.key.pem ]; then
                ## Create private key for site
                openssl genrsa \
                    -out /var/www/$DOMAIN/ssl/private/$DOMAIN.key.pem 2048 \
                     > /dev/null 2>&1

                ## Create CSR
                openssl req \
                    -config /vagrant/openssl.conf \
                    -key /var/www/$DOMAIN/ssl/private/$DOMAIN.key.pem \
                    -new -sha256 -out /var/www/$DOMAIN/ssl/private/$DOMAIN.csr.pem \
                    -subj "/CN=$DOMAIN" \
                    > /dev/null 2>&1
q
                ## Self sign cert
                openssl ca -batch \
                    -config <(echo "$openssl
                    [ san_self_signed ]
                    subjectAltName = DNS:$DOMAIN, DNS:localhost
                    ") \
                    -extensions san_self_signed -days 375 -notext -md sha256 \
                    -in /var/www/$DOMAIN/ssl/private/$DOMAIN.csr.pem \
                    -out /var/www/$DOMAIN/ssl/certs/$DOMAIN.cert.pem \
                    > /dev/null 2>&1
            fi

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
