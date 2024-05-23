# 베이스 이미지
FROM wordpress:5.8-apache

ENV WORDPRESS_DB_HOST=db-hs1.cluster-cdy6a8acgxti.ap-northeast-2.rds.amazonaws.com
ENV WORDPRESS_DB_PASSWORD=Amazon1!
ENV WORDPRESS_DB_USER=admin
ENV WORDPRESS_DB_NAME=dbhouse

# 포트 개방
EXPOSE 80

# 볼륨 마운트 정의
VOLUME ["/var/www/html"]


# 필요한 패키지 설치
#RUN yum update -y && \
    yum install -y httpd mysql php php-mysqlnd php-fpm php-json tar gzip curl

# Apache와 PHP-FPM 설정 파일 수정
#COPY httpd.conf /etc/httpd/conf/httpd.conf
#COPY php-fpm.conf /etc/php-fpm.conf

# WordPress 다운로드 및 설치
#RUN curl -O https://wordpress.org/wordpress-5.8.tar.gz && \
    tar -zxvf wordpress-5.8.tar.gz -C /var/www/html/ && \
    chown -R apache:apache /var/www/html && \
    chmod -R 755 /var/www/html
