
### :Protobuf ��װ
>
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
		
### :Protobuf ʹ��   ��������� hbase/java/

- һ���绰 ������  ���

##### ������ �Ǹ�Phone.java�ļ� 
	
	��Ŀ¼��


##### phoneCase �Ƕ� ���ݲ��� �����װ���� ������װ����
> 
	ע�⡣ put(rowKey)
			put.add("cf".getBytes(),"dnum".getBytes(),dnum.getBytes());
			get(rowkey);
			table.put(puts); 
			table.get(get);
	
	Scan scan = new Scan()
	scan.setStartrow("");   // �� row�� ����  ��ʼ��Χ 
	scan.setStopRow("");   //   ֹͣ��Χ 
	
	
	/**
	 ��ѯ ָ�� column�е����� 
	FiltersList lis  = new FiltersList(FiltersList.Operator.MUST_PASS_ALL);  ///  ����������������
	SingleColumnValueFilter filter= new SingleColumnValueFilter(familyName.getBytes(), 
	"type".getBytes(),  CompareOp.EQUAL, "0".getBytes());  // ֻ�� colum�е� type == 0�ű�ȡ��
	PrefixFilter filter2 =new PrefixFilter("15894311313".getBytes());
	
	lis.addFilter(filter);  // ���뵽 ���������С�
	lis.addFilter(filter);
	
	Scan scan = new Scan();
	scan.setFillter(filters);
	ResultScanner scanner = table.getScanner(scan);
	for��Result result: scanner�� {
			System.out.print(Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "dnum".getBytes()))));
			System.out.print("=="+Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "type".getBytes()))));
			System.out.print("=="+Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "length".getBytes()))));
			System.out.println("=="+Bytes.toString(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), "date".getBytes()))));
	}
	scanner.close();


>   ���� ���� 

	Phone.PhoneDetail.Builder phoneDetail = Phone.PhoneDetail.newBuilder();
	phoneDetail.setDate(date);
	phoneDetail.setDnum(dnum);
	phoneDetail.setLength(length);
	phoneDetail.setType(type);
	
	String rowkey = phoneNumber+"_"+String.valueOf(Long.MAX_VALUE-sdf.parse(date).getTime());
	Put put = new Put(rowkey.getBytes());
	put.add("cf".getBytes(),"phoneDetail".getBytes(),phoneDetail.build().toByteArray());
	puts.add(put);

>  ��1000�� phoneDetail ��װ�� һ�� DayOfPhone�� 
		���� 
	String rowkey = phoneNumber+"_"+String.valueOf(Long.MAX_VALUE-sdf.parse("20181225000000").getTime());
	Phone.DayOfPhone.Builder dayof = Phone.DayOfPhone.newBuilder();
	for(int j =0;j<1000;j++){
		// ���� 
		String dnum = getPhone("177");
		String length = String.valueOf(r.nextInt(99));// ͨ��ʱ��
		String type = String.valueOf(r.nextInt(2));
		String date = getDate2("20181225");
		
		Phone.PhoneDetail.Builder phoneDetail = Phone.PhoneDetail.newBuilder();
		phoneDetail.setDate(date);
		phoneDetail.setDnum(dnum);
		phoneDetail.setLength(length);
		phoneDetail.setType(type);
		dayof.addDayPhone(phoneDetail);
	}
	Put put = new Put(rowkey.getBytes());
	put.add("cf".getBytes(), "day".getBytes(), dayof.build().toByteArray());
	puts.add(put);   //  10 ֻ��10 ������ rowkey 

	table.put(puts);
		
		
	��ȡ���� 
		Get get = new Get("15893304049_9223370522077571807".getBytes());
	Result result = table.get(get);
	PhoneDetail phoneDetail = Phone.PhoneDetail.parseFrom(CellUtil.cloneValue(result.getColumnLatestCell("cf".getBytes(), 
			"phoneDetail".getBytes())));
	System.out.println(phoneDetail);	











