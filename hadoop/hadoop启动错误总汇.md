

hdfs ->[node06 (node07 namenode] (node08 node09 datanode)-<zookeeper
### ��Ⱥ���ִ���: 
>
	1. node06  node07����  standly״̬ 
		��� 1�� �ڻỰ�� ִ��jps,�鿴�����ڵ��״̬,����û��zkfc(DFSZKFailoverController)
				ԭ���� zookeeper��Ⱥ��������֮�� û�и�ʽ��zkfc ���Ӷ�û�п���hdfs 
				���� �������ڵ㶼����standly״̬����Ⱥ���ڲ�����״̬
			�취�� stop-all.sh  �������� dfs �� yarn js�ڴ˲鿴 jps	
				 �Ự��ִ�� zkServer.sh start node07 node08 node09 ����QuorumPeerMain
				 node06���ڵ���ִ�� start-dfs.sh start-yarn.sh node08 node09��yarn-daemon.sh start resourcemanager
>	
	2. node06 ��standly node07 ��active ״̬��
		��� 1�� �Ự�£�ִ�� jsp���� zkfc���� ���� 
				ԭ���ǣ�node06 ��ĳ��ʱ����zkfc��Ϊ�ҵ��� �����԰�namenode�Ŀ���Ȩ�����˱��ýڵ�node07
			��� �� ��node07��ִ�� hadoop-daemon.sh stop namenode ͣ������
			��������� ���� node06:60070 ״̬ ��standly׼��Ϊ active�����û���򣬶�ˢ�¼��Ρ�
				���ʵ�ڲ��У��Ͱ�hdfs yarn����һ��  ��סҪ���� date һ�� �� ��date -s "2019-08-19 31:36"

>				
	�Ựzookeeper.sh 			
	node06 start-dfs.sh start-yarn.sh 
	node08 9  yarn-daemon.sh start resourcemanager
	
	node06 start-hbase.sh 
	node06 hbase shell