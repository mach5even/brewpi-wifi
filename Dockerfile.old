FROM phusion/baseimage:0.9.18
 
MAINTAINER mac@mach5even.com

ENV DEBIAN_FRONTEND noninteractive
ENV AVR=192.168.1.126:23 
ENV TZ=America/Montreal

RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold"
RUN apt-get install --no-install-recommends -y apache2 libapache2-mod-php5 php5-cli php5-common php5-cgi git git-core php5  python-serial python-simplejson python-configobj python-psutil python-git python-pip python-dev build-essential
RUN pip install pyserial --upgrade
RUN pip install psutil --upgrade
RUN useradd -m -k /dev/null -u 99 -g 100 -G www-data,dialout brewpi && sh -c 'echo "brewpi:`openssl rand -base64 32`" | chpasswd'
RUN usermod -a -G www-data brewpi
RUN usermod -a -G users brewpi
RUN chown -R www-data:www-data /var/www
RUN chown -R brewpi:users /home/brewpi
RUN find /home/brewpi -type f -exec chmod g+rwx {} \;
RUN find /home/brewpi -type d -exec chmod g+rwxs {} \;
RUN find /var/www -type d -exec chmod g+rwxs {} \;
RUN find /var/www -type f -exec chmod g+rwx {} \;
RUN sed -i 's#DocumentRoot /var/www/html#DocumentRoot /var/www#' /etc/apache2/sites-available/000-default.conf
RUN sudo -u brewpi git clone --branch 0.3.8 --depth 1  https://github.com/BrewPi/brewpi-script /home/brewpi && \
    rm -rf /var/www/* && sudo -u www-data git clone https://github.com/BrewPi/brewpi-www /var/www 
RUN sed -i 's#inWaiting = ser.inWaiting()#inWaiting = ser.readline()#' /home/brewpi/brewpi.py
RUN sed -i 's#newData = ser.read(inWaiting)#newData = inWaiting#' /home/brewpi/brewpi.py
RUN sed -i 's#ser = serial.Serial(port, baudrate=baud_rate, timeout=time_out)#ser = serial.serial_for_url(port, baudrate=baud_rate, timeout=1)#' /home/brewpi/BrewPiUtil.py
RUN sed -i 's#brewpi:brewpi#brewpi:users#' /home/brewpi/utils/fixPermissions.sh
RUN sed -i 's#ser.setTimeout(1)#\# ser.setTimeout(1)#' /home/brewpi/brewpiVersion.py
RUN sed -i 's#ser.setTimeout(oldTimeOut) \# restore previous serial timeout value#\# ser.setTimeout(oldTimeOut) \# restore previous serial timeout value#' /home/brewpi/brewpiVersion.py
RUN chown -R brewpi:users /home/brewpi/settings
RUN chmod +x /home/brewpi/*.py
RUN chmod +x /home/brewpi/utils/*.sh
RUN echo 'scriptPath = /home/brewpi/' > /home/brewpi/settings/config.cfg
RUN echo 'wwwPath = /var/www/' >> /home/brewpi/settings/config.cfg
RUN echo 'boardType = standard' >> /home/brewpi/settings/config.cfg
RUN echo 'beerName = default' >> /home/brewpi/settings/config.cfg
RUN echo 'interval = 120.0' >> /home/brewpi/settings/config.cfg
RUN echo 'dataLogging = active' >> /home/brewpi/settings/config.cfg
RUN echo 'port = socket://example' >> /home/brewpi/settings/config.cfg
RUN echo 'profileName = default' >> /home/brewpi/settings/config.cfg
RUN echo "TZ=$TZ" >/etc/cron.d/brewpi
RUN echo "* * * * * brewpi python /home/brewpi/brewpi.py --checkstartuponly --dontrunfile /home/brewpi/brewpi.py 1>/dev/null 2>>/home/brewpi/logs/stderr.txt; [ \$? != 0 ] && python -u /home/brewpi/brewpi.py 1>/home/brewpi/logs/stdout.txt 2>>/home/brewpi/logs/stderr.txt &" >>/etc/cron.d/brewpi
RUN echo "#!/bin/bash" >/root/runbrewpi.sh
RUN echo "/sbin/my_init &" >>/root/runbrewpi.sh
RUN echo "sed -i 's#^port.*#port = socket://'\"\$AVR\"'#' /home/brewpi/settings/config.cfg" >>/root/runbrewpi.sh
RUN echo "sed -i 's#^altport.*#altport = socket://'\"\$AVR\"'#' /home/brewpi/settings/config.cfg" >>/root/runbrewpi.sh
RUN echo "sed -i 's#^TZ.*#TZ='\"\$TZ\"'#' /etc/cron.d/brewpi" >>/root/runbrewpi.sh
RUN echo "/usr/sbin/apache2ctl -D FOREGROUND" >>/root/runbrewpi.sh
RUN /home/brewpi/utils/fixPermissions.sh
RUN chmod +x /root/runbrewpi.sh

VOLUME /home/brewpi/data
VOLUME /var/www/data
VOLUME /home/brewpi/settings
 
EXPOSE 80
 
ENTRYPOINT /root/runbrewpi.sh
