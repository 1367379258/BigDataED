1.    tar  -zxf protobuf-.......
2. yum groupinstall "Development tools"
3. ./config......   Ĭ�ϰ���װ�� usr/local
4. make && make  install 

> 
	package com.bjsxt.hbase;
	message PhoneDetail
	{

		required string dnum = 1;
		required string length = 2;
		required string type = 3;
		required string date = 4;
	}
	message DayOfPhone
	{
		repeated PhoneDetail DayPhone = 1;
	}

		vi phone.proto ����������� �Ž�ȥ 

		/usr/local/bin/protoc phone.proto --java_out=/root/

		ftps ��java�ŵ� eclipese


		insert3 ���� phone3��