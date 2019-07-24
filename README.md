# AWS Lambda Custom Runtime for PHP
This repository has precompiled PHP 7.3.7 binary file in [`/bin`](https://github.com/maksymilian-lewicki/php-lambda-example/tree/master/bin) directory.

[Here](#how-to-compile-own-php) is instruction how to compile own PHP binary file

[This](https://aws.amazon.com/blogs/apn/aws-lambda-custom-runtime-for-php-a-practical-example/) is official tutorial how to create own custom AWS Lambda runtime for PHP.
## How to run custom Lambda runtime for PHP
1. Clone this repository 
	```
	git clone https://github.com/maksymilian-lewicki/php-lambda-example.git
	```
2. Install composer dependencies
	```bash
	composer install
	```
3. Make zip files
	```bash
	zip -r runtime.zip bin bootstrap
	zip -r vendor.zip vendor/
	zip hello.zip src/hello.php
	zip goodbye.zip src/goodbye.php
	```
4. Create IAM role with name `LambdaPhpExample` and attach `AWSLambdaBasicExecutionRole` policy. Make note of the Role.Arn value (e.g. `arn:aws:iam::XXXXXXXXXXXX:role/LambdaPhpExample`), which you’ll need for the next steps.
5. Create `runtime` and `vendor` layers:
	```bash
	aws lambda publish-layer-version \
	--layer-name php-example-runtime \
	--zip-file fileb://runtime.zip \
	--region eu-west-1
	```

	```bash
	aws lambda publish-layer-version \
	--layer-name php-example-vendor \
	--zip-file fileb://vendor.zip \
	--region eu-west-1
	```
	Make note of each command’s LayerVersionArn output value (e.g. `arn:aws:lambda:eu-west-1:XXXXXXXXXXXX:layer:php-example-runtime:1`), which you’ll need for the next steps.
6. Create our two functions
	```bash
	aws lambda create-function \
	--function-name php-example-hello \
	--handler hello \
	--zip-file fileb://./hello.zip \
	--runtime provided \
	--role "arn:aws:iam::XXXXXXXXXXXX:role/LambdaPhpExample" \
	--region eu-west-1 \
	--layers "arn:aws:lambda:eu-west-1:XXXXXXXXXXXX:layer:php-example-runtime:1" \
	"arn:aws:lambda:eu-west-1:XXXXXXXXXXXX:layer:php-example-vendor:1"
	```

	```bash
	aws lambda create-function \
	--function-name php-example-goodbye \
	--handler goodbye \
	--zip-file fileb://./goodbye.zip \
	--runtime provided \
	--role "arn:aws:iam::XXXXXXXXXXXX:role/LambdaPhpExample" \
	--region eu-west-1 \
	--layers "arn:aws:lambda:eu-west-1:XXXXXXXXXXXX:layer:php-example-runtime:1" \
	"arn:aws:lambda:eu-west-1:XXXXXXXXXXXX:layer:php-example-vendor:1"
	```
7. Invoke them for test
	```bash
	aws lambda invoke \
	--function-name php-example-hello \
	--region eu-west-1 \
	--log-type Tail \
	--query 'LogResult' \
	--output text \
	--payload '{"name": "World"}' hello-output.txt | base64 --decode
	```

	```bash
	aws lambda invoke \
	--function-name php-example-goodbye \
	--region eu-west-1 \
	--log-type Tail \
	--query 'LogResult' \
	--output text \
	--payload '{"name": "World"}' goodbye-output.txt | base64 --decode
	```
	A quick `cat` of the `hello-output.txt` and `goodbye-output.txt` files will also reveal the `Hello, World!` and `Goodbye, World!` output, respectively.
## How to compile own PHP:
1. Launch an EC2 instance [amzn-ami-hvm-2018.03.0.20181129-x86_64-gp2](https://console.aws.amazon.com/ec2/v2/home#Images:visibility=public-images;search=amzn-ami-hvm-2018.03.0.20181129-x86_64-gp2). You can find latest AMI on [this site](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html).
2. Update packages and install needed compilation dependencies
	```bash
	sudo yum update -y
	sudo yum install autoconf bison gcc gcc-c++ libcurl-devel libxml2-devel -y
	```
3. Compile OpenSSL
	```bash
	curl -sL http://www.openssl.org/source/openssl-1.0.1k.tar.gz | tar -xvz
	cd openssl-1.0.1k
	./config && make && sudo make install
	cd ~
	```
4. Download the PHP source
	```bash
	curl -sL https://github.com/php/php-src/archive/php-7.3.7.tar.gz | tar -xvz
	cd ~/php-src-php-7.3.7
	```
5. Compile PHP with OpenSSL support
	```bash
	./buildconf --force
	./configure --prefix=/home/ec2-user/php-7.3-bin/ --with-openssl=/usr/local/ssl --with-curl --with-zlib
	make install
	```
6. Binary file `~/php-7.3-bin/bin/php` can be used as our runtime in AWS Lamba

To verify if compilation of PHP went well run command:
```bash
/home/ec2-user/php-7.3-bin/bin/php -v
```
You should see some variation of the following, depending on the version you installed:
```bash
PHP 7.3.7 (cli) (built: Jul 22 2019 09:16:39) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.7, Copyright (c) 1998-2018 Zend Technologies
```
