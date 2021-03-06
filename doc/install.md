## Install

This guide is tested on Debian/Ubuntu systems

### Packages
    # Update packages system
    aptitude update
    aptitude upgrade

    # Neccesary packages
    aptitude install sudo git nodejs

    #Add user
    sudo adduser --disabled-login --gecos 'TurboBil' turbobil
    #Add user turbobil to sudoers
    echo "turbobil ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/turbobil


### Database

For install with MySQL check the [Guide for MySQL] (db_mysql.md)

    # Install PostgreSQL database
    sudo aptitude install -y postgresql postgresql-client libpq-dev python-psycopg2

    # Access to PostgreSQL
    sudo -u postgres psql -d template1
    # Create a user
    template1=# CREATE USER turbobil CREATEDB;
    # Create the database
    template1=# CREATE DATABASE turbobil OWNER turbobil;
    # Exit session on database
    template1=# \q


Add this line

    host    turbobil        turbobil        127.0.0.1/32            trust

before

    host    all             all             127.0.0.1/32            md5


on */etc/postgresql/9.3/main/pg_hba.conf*

#### Restart service PostgreSQL

    /etc/init.d/postgresql restart


### Repository and files
    # Go to home user
    cd /home/turbobil
    # Clone repository
    sudo -u turbobil -H git clone https://github.com/roramirez/turbobil.git -b master tbil
    cd tbil
    sudo -u turbobil -H git submodule init
    sudo -u turbobil -H git submodule update


The project is develop in Ruby 2.1.2


  Install Ruby by RVM

    su turbobil
    \curl -sSL https://get.rvm.io | bash -s -- --ruby=2.1.2
    source ~/.rvm/scripts/rvm
    rvm --default use 2.1.2


  Install Ruby from Source

    # Required packages to compile Ruby
    sudo aptitude install  build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server redis-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate python-docutils pkg-config cmake

    # tmp dir
    mkdir /tmp/src && cd /tmp/src
    # download source version 2.1.2
    curl -L --progress ftp://ftp.ruby-lang.org/pub/ruby/2.1/ruby-2.1.2.tar.gz | tar xz
    cd ruby-2.1.2
    ./configure --disable-install-rdoc
    make
    sudo make install

#### Config enviroment
    su turbobil
    sudo gem install bundler --no-ri --no-rdoc
    cd rails/


    # For PostgreSQL
    sudo bundle install --without mysql

    # For MySQL
    sudo bundle install --without postgresql

    # PostgreSQL Config database
    cp tbil/rails/config/database.yml.postgresql tbil/rails/config/database.yml

    # MySQL config
    cp tbil/rails/config/database.yml.mysql tbil/rails/config/database.yml
    # Update password in config/database.yml
    # Change 'changeme' with the value you have given to $password


    #initialized db
    rake db:migrate --trace RAILS_ENV=production

    #assets
    rake assets:precompile RAILS_ENV=production

    #set variable env
    export RAILS_ENV=production
    export SECRET_KEY_BASE=$(rake secret)

    #load fixtures
    rake db:fixtures:load

    #run app
    rails s

### Start
and now you have two app
admin http://IP_MACHINE:3000/admins
  - Email: admin@example.com
  - Password: password

For customers http://IP_MACHINE:3000/customer


### Asterisk

Now Turbobil working with Asterisk

If dont have Asterisk installed check this mini guide 


#### Install dependences

    aptitude install build-essential gcc g++ automake autoconf libtool make \
         libncurses5-dev flex bison patch libtool autoconf \
         linux-headers-$(uname -r) libxml2-dev cmake


If you will install Asterisk on version 12 add these dependences
    aptitude install libjansson-dev uuid-dev


#### Compiling and install
Will do this as user root

    cd /usr/src
    wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-1.8.30.0.tar.gz
    tar xvfz asterisk-1.8.30.0.tar.gz
    cd asterisk-1.8.30.0
    ./configure
    make
    make install
    make config
    make samples

#### Configuration
    cp /home/turbobil/tbil/agi_ast/exten_tbill.conf /etc/asterisk
    ln -s /home/turbobil/tbil/agi_ast/ /var/lib/asterisk/agi-bin/tbil

    # PostgreSQL Config database
    cp /home/turbobil/tbil/agi_ast/config.ini-dist /home/turbobil/tbil/agi_ast/config.ini

    # MySQL config
    cp /home/turbobil/tbil/agi_ast/config.ini-dist-mysql /home/turbobil/tbil/agi_ast/config.ini
    # Update password in agi_ast/config.ini
    # Change 'changeme' with the value you have given to $password

    echo "#include \"/home/turbobil/tbil/rails/config/sips/*.account\"" >> /etc/asterisk/sip.conf
    echo "#include \"/home/turbobil/tbil/rails/config/sips/*.provider\"">> /etc/asterisk/sip.conf
    echo "#include \"exten_tbill.conf\"" >> /etc/asterisk/extensions.conf

    /etc/init.d/asterisk start
