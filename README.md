# php56pthread
PHP 5.6 Pthreads

yum install php-pear
pear install PEAR-1.10.1
pear install PHP_Archive-0.12.0

yum install bzip2-devel && yum install curl && yum install curl-devel && yum install libpng-devel && yum install libc-client-devel && yum install postgresql-devel && yum install sqlite-devel && yum install libxslt-devel && yum install libxml2-devel && yum install libmcrypt-devel.x86_64 && yum install libtidy libtidy-devel && yum install libjpeg-devel
yum install mysql-devel && yum install freetype-devel && yum install libicu-devel

algum outro pacote que der erro veja aqui
http://crybit.com/20-common-php-compilation-errors-and-fix-unix/

cp php56pthread.zip /opt/
cd /opt/
unzip php56pthread.zip
cd php-5.6.16/
rm configure
./buildconf --force 
./configure '--prefix=/opt/php-5.6.16' '--with-pdo-pgsql' '--with-zlib-dir' '--with-freetype-dir' '--enable-mbstring' '--with-libxml-dir=/usr' '--enable-soap' '--enable-calendar' '--with-curl' '--with-mcrypt' '--with-gd' '--with-pgsql' '--disable-rpath' '--enable-inline-optimization' '--with-bz2' '--with-zlib' '--enable-sockets' '--enable-sysvsem' '--enable-sysvshm' '--enable-pcntl' '--enable-mbregex' '--enable-exif' '--enable-bcmath' '--with-mhash' '--enable-zip' '--with-pcre-regex' '--with-mysql' '--with-pdo-mysql' '--with-mysqli' '--with-jpeg-dir=/usr' '--with-png-dir=/usr' '--enable-gd-native-ttf' '--with-openssl' '--with-fpm-user=www-data' '--with-fpm-group=www-data' '--with-libdir=/lib' '--enable-ftp' '--with-kerberos' '--with-gettext' '--with-xmlrpc' '--with-xsl' '--enable-opcache' '--enable-fpm' '--enable-debug' '--enable-maintainer-zts' '--enable-pthreads' '--enable-json' '--enable-apcu' '--enable-cgi'

make clean
make
make install

Criar arquivo que inicializar php-fpm
vim /etc/init.d/php-5.6.16-fpm
conteúdo do arquivo

```shell
                #! /bin/sh
                ### BEGIN INIT INFO
                # Provides:          php-5.6.16-fpm
                # Required-Start:    $all
                # Required-Stop:     $all
                # Default-Start:     2 3 4 5
                # Default-Stop:      0 1 6
                # Short-Description: starts php-5.6.16-fpm
                # Description:       starts the PHP FastCGI Process Manager daemon
                ### END INIT INFO
                php_fpm_BIN=/opt/php-5.6.16/sbin/php-fpm
                php_fpm_CONF=/opt/php-5.6.16/etc/php-fpm.conf
                php_fpm_PID=/opt/php-5.6.16/var/run/php-fpm.pid
                php_opts="--fpm-config $php_fpm_CONF"
                wait_for_pid () {
                        try=0
                        while test $try -lt 35 ; do
                                case "$1" in
                                        'created')
                                        if [ -f "$2" ] ; then
                                                try=''
                                                break
                                        fi
                                        ;;
                                        'removed')
                                        if [ ! -f "$2" ] ; then
                                                try=''
                                                break
                                        fi
                                        ;;
                                esac
                                echo -n .
                                try=`expr $try + 1`
                                sleep 1
                        done
                }
                case "$1" in
                        start)
                                echo -n "Starting php-fpm "
                                $php_fpm_BIN $php_opts
                                if [ "$?" != 0 ] ; then
                                        echo " failed"
                                        exit 1
                                fi
                                wait_for_pid created $php_fpm_PID
                                if [ -n "$try" ] ; then
                                        echo " failed"
                                        exit 1
                                else
                                        echo " done"
                                fi
                        ;;
                        stop)
                                echo -n "Gracefully shutting down php-fpm "
                                if [ ! -r $php_fpm_PID ] ; then
                                        echo "warning, no pid file found - php-fpm is not running ?"
                                        exit 1
                                fi
                                kill -QUIT `cat $php_fpm_PID`
                                wait_for_pid removed $php_fpm_PID
                                if [ -n "$try" ] ; then
                                        echo " failed. Use force-exit"
                                        exit 1
                                else
                                        echo " done"
                                       echo " done"
                                fi
                        ;;
                        force-quit)
                                echo -n "Terminating php-fpm "
                                if [ ! -r $php_fpm_PID ] ; then
                                        echo "warning, no pid file found - php-fpm is not running ?"
                                        exit 1
                                fi
                                kill -TERM `cat $php_fpm_PID`
                                wait_for_pid removed $php_fpm_PID
                                if [ -n "$try" ] ; then
                                        echo " failed"
                                        exit 1
                                else
                                        echo " done"
                                fi
                        ;;
                        restart)
                                $0 stop
                                $0 start
                        ;;
                        reload)
                                echo -n "Reload service php-fpm "
                                if [ ! -r $php_fpm_PID ] ; then
                                        echo "warning, no pid file found - php-fpm is not running ?"
                                        exit 1
                                fi
                                kill -USR2 `cat $php_fpm_PID`
                                echo " done"
                        ;;
                        *)
                                echo "Usage: $0 {start|stop|force-quit|restart|reload}"
                                exit 1
                        ;;
                esac
...


Em /opt/php-5.6.16/etc/php-fpm.conf alterar usuário e grupo para os que executam o seu webserver ex.: apache ou nginx
neste caso usuário e grupos do apache www-data
user = www-data
group = www-data

Inicie o arquivo
/etc/init.d/php-5.6.16-fpm start

Para habilitar no Nginx
cd /etc/nginx/sites-enabled
vim default
mudar porta do php no nginx para 8999
colocar o arquivo /etc/init.d/php-5.6.16-fpm para iniciar com o sistema
