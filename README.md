# AWS Lambda Custom Runtime for PHP
[Here](https://aws.amazon.com/blogs/apn/aws-lambda-custom-runtime-for-php-a-practical-example/) is official tutorial how to create own custom AWS Lambda runtime for PHP.
## How to compile own PHP:
1. Launch an EC2 instance [amzn-ami-hvm-2018.03.0.20181129-x86_64-gp2](https://console.aws.amazon.com/ec2/v2/home#Images:visibility=public-images;search=amzn-ami-hvm-2018.03.0.20181129-x86_64-gp2). You can find latest AMI on [this site](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html).
2. Update packages and install needed compilation dependencies
	```
	sudo yum update -y
	sudo yum install autoconf bison gcc gcc-c++ libcurl-devel libxml2-devel -y
	```
3. Compile OpenSSL
	```
	curl -sL http://www.openssl.org/source/openssl-1.0.1k.tar.gz | tar -xvz
	cd openssl-1.0.1k
	./config && make && sudo make install
	cd ~
	```
4. Download the PHP source
	```
	curl -sL https://github.com/php/php-src/archive/php-7.3.7.tar.gz | tar -xvz
	cd ~/php-src-php-7.3.7
	```
5. Compile PHP with OpenSSL support
	```
	./buildconf --force
	./configure --prefix=/home/ec2-user/php-7.3-bin/ --with-openssl=/usr/local/ssl --with-curl --with-zlib
	make install
	```
6. Binary file `~/php-7.3-bin/bin/php` can be used as our runtime in AWS Lamba

To verify if compilation of PHP went well run command:
```
/home/ec2-user/php-7.3-bin/bin/php -v
```
You should see some variation of the following, depending on the version you installed:
```
PHP 7.3.7 (cli) (built: Jul 22 2019 09:16:39) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.7, Copyright (c) 1998-2018 Zend Technologies
```
