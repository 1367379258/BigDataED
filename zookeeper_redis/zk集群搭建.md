1.3�ڵ� java ��װ 

2.���м�Ⱥ�ڵ㴴��Ŀ¼: mkdir opt/sxt  

3.zkѹ������ѹ������·����:��
	#	tar xf zookeeper-3.4.6.tar.gz -C /opt/sxt/

4.����confĿ¼������zoo_sample.cfg zoo.cfg ������
   dataDir����Ⱥ�ڵ㡣
server.1=node07:2888:3888
server.2=node08:2888:3888
server.3=node08:2888:3888
5.���ڵ����û������������ַ� ZOOKEEPER_PREFIX������ģʽ��ȡprofile 

6. ������ /var/sxt/zkĿ¼���������Ŀ¼ �ֱ����1,2��3 ���ļ� myid
	echo 1 > /var/sxt/zk/myid
	...

7. ��������zkServer.sh start ��Ⱥ


8.�����ͻ��� help����鿴














