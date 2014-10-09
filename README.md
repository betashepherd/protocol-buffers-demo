protocol-buffers-demo
====================

A protocol buffers demo with PHP.

```shell
git clone https://github.com/betashepherd/protobuf.git

./configure --prefix=/usr/local/protobuf && make && make install

git clone https://github.com/betashepherd/php-protocolbuffers.git

phpize && ./configure --with-php-config=/path/to/php-config && make && make install

git clone https://github.com/betashepherd/protoc-gen-php.git

curl -sS https://getcomposer.org/installer | php

php composer.phar install

/usr/local/protobuf/bin/protoc --plugin=protoc-gen-php/bin/protoc-gen-php --php_out=proto-php-files -I. *.proto
```

```php
<?php

require 'autoload.php';

$ListRequest = new el\ListRequest ();
$ListRequest->setCheckInDate ( 1412265600 );
$ListRequest->setCheckOutDate ( 1412352000 );
$ListRequest->setRegionId ( 178293 );
$ListRequest->setRankType(0);
$ListRequest->setHotelNum(10);

$AsRequest = new el\AsRequest();
$AsRequest->setSearchType(2);
$AsRequest->setListReq($ListRequest);

$request = $AsRequest->serializeToString();
$head = pack ( "nnIa16III", 1, 1, 1, "abc", 1, 0, strlen ( $request ) );

$host = '192.168.14.171'; $port = 5312;

$socket = socket_create ( AF_INET, SOCK_STREAM, SOL_TCP );

socket_connect ( $socket, $host, $port );
socket_write ( $socket, $head, strlen($head));
socket_write ( $socket, $request, strlen($request));

$response_head = socket_read($socket, strlen($head));
$head_data = unpack ( "n/n/I/a16/I/I/Ibody_len", $response_head);

$body_len = $head_data['body_len'];


if($body_len !== 0) {
	
	$buffer = '';
    socket_recv($socket, $buffer, $body_len, MSG_WAITALL);
        
    $AsResponse = \el\AsResponse::parseFromString($buffer);
    $hotels = $AsResponse->getListResp()->getHotelList();
    
    
     foreach ($hotels as $hotel) {
     	print_r($hotel);exit;
    	#echo $hotel->hotel_id. PHP_EOL;
    } 
    
} else {
    print 'null response';
}
```
