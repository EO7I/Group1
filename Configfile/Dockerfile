# 베이스 이미지로부터 시작
FROM amazonlinux:2

# 필요한 패키지 설치
RUN yum update -y && \
    yum install -y httpd mysql php php-mysqlnd php-fpm php-json tar gzip curl

# Apache와 PHP-FPM 설정 파일 수정
COPY httpd.conf /etc/httpd/conf/httpd.conf
COPY php-fpm.conf /etc/php-fpm.conf

# WordPress 다운로드 및 설치
RUN curl -O https://wordpress.org/latest.tar.gz && \
    tar -zxvf latest.tar.gz -C /var/www/html/ && \
    chown -R apache:apache /var/www/html/wordpress && \
    chmod -R 755 /var/www/html/wordpress

# WordPress의 index.php 파일을 사용하기 위해 복사
COPY index.php /var/www/html/wordpress/

# Apache 포트 개방
EXPOSE 80

# Apache와 PHP-FPM 실행
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
