# α�ֲ�ʽ ��single node setup��
---------------------------


## һ�� ��װjdk�����û�������������

> ��ssh  localhost        /  node06      �޸�/etc/hosts         �к���ڶ���exit�˳�

>����Կ
	ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
	cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys


> hadoop����װ�����û��䣺hadoop-2.6.5.tar.gz
export HADOOP_HOME=/opt/sxt/hadoop-2.6.5
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

* Hadoop�ĵڶ���JAVA_HOME ������������

> vi hadoop-env.sh    �޸��ļ���java������
> vi mapred-env.sh
> vi yarn-env.sh

* ����core-site.xml
 
> vi core-site.xml

    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://node06:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>     // ��ʱ�ļ��� ���λ��
        <value>/var/sxt/hadoop/local</value>
    </property>


* ����hdfs-site.xml
 > vi  hdfs-site.xml
 
    <property>
        <name>dfs.replication</name>    // ��������
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>   // �ڶ��ڵ� ����nameNode
        <value>node06:50090</value>
    </property>
	
* ����slaves�ļ�
> vi slaves node06
			
	��ʽ��hdfs
	hdfs namenode -format  (ֻ�ܸ�ʽ��һ�Σ��ٴ�������Ⱥ��Ҫִ�У�
	������Ⱥ
	start-dfs.sh

	��ɫ���̲鿴��jps
	������ hdfs 
		   hdfs dfs 	

ss -nal ���鿴hadoop״̬����
�鿴web UI: IP:50070

     ����Ŀ¼��hdfs dfs  -mkdir -p  /user/root 
		
     �鿴Ŀ¼:  hdfs dfs -ls   /
	
     �ϴ��ļ��� hdfs dfs -put  hadoop-2.6.5.tar.gz   /user/root    blockһ��Ϊ 128 > 128 �ͻ���/var/sxt/hadoop���µ�datanode���´���һ��block       ���ǵ�½ node06:50070 ���Բ鿴 ��������		
    
      ֹͣ��Ⱥ��stop-dfs.sh



# ȫ�ֲ���װ
---------------------------------------------
##  �� �� ǰ��׼����

jdk
hostname
hosts
date
��ȫ����
firewall
windows ����ӳ��

�ڵ㣺 node06/07/08/09
ȫ�ֲ����䷽����

>					NN	SNN	DN
>		NODE06		*
>		NODE07			*	*	
>		NODE08				*
>		NODE09				*


> �ڵ�״̬��
	node06�� α�ֲ�
	node07/08/09 :  ip�������
�������ڵ�ͨѶ��hosts��


* �� ����ʱ��ͬ����date -s ��xxxx-x-xx xx:xx:xx��


> ��Կ�ַ��� 
��ÿ���ڵ��ϵ�¼һ���Լ�������.sshĿ¼
��node06��node07/node08/node09�ַ���Կ (��Կ������Ҫ�仯)
scp  id_dsa.pub   node07:`pwd`/node06.pub

>���ڵ��node01�Ĺ�Կ׷�ӵ���֤�ļ��            ls -a�鿴�����ļ�
cat ~/node06.pub  >> ~/.ssh/authorized_keys


> 07/node08/node09��װjdk������node06�ַ�profile�������ڵ㣬���ض������ļ�

> scp �ַ�hadoop�������2.6.5 �������ڵ�


* copy node06 �µ� hadoop Ϊ hadoop-local ������ű�ֻ���ȡhadoopĿ¼��

> [root@node06 etc]# cp -r hadoop   hadoop-local
> ����core-site.xml
> ����hdfs-site.xml
> ����slaves
> node07
> node08
> node09

> root@node06 opt]# scp -r sxt/ node07:`pwd`
�ַ�sxt������07,08��09�ڵ�

��ʽ����Ⱥ��hdfs namenode -format

������Ⱥ��start-dfs.sh
Jps  �鿴���ڵ�����������


## HA 
-------------------
>
		    NN1 	NN2		DN		ZK		ZKFC	JNN
	NODE06		*							*		*
	NODE07			*		*		*		*		*
	NODE08					*		*				*
	NODE09					*		*


* ����nn�ڵ�����Կ
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

-------------------------
>
	����hadoop-2.6.5/etcĿ¼ (����ͨ��������cd $HADOOP_HOME)
	����hadoop Ϊ hadoop-full



hdfs.xml
---------------------------------------

* ȥ��snn������
>

	 <property>
					  <name>dfs.namenode.secondary.http-address</name>
				  <value>node07:50090</value>
	 </property>


* ���ӣ�
>

	<property>
	  <name>dfs.nameservices</name>
	  <value>mycluster</value>
	</property>
	<property>
	  <name>dfs.ha.namenodes.mycluster</name>
	  <value>nn1,nn2</value>
	</property>
	<property>
	  <name>dfs.namenode.rpc-address.mycluster.nn1</name>
	  <value>node06:8020</value>
	</property>
	<property>
	  <name>dfs.namenode.rpc-address.mycluster.nn2</name>
	  <value>node07:8020</value>
	</property>
	<property>
	  <name>dfs.namenode.http-address.mycluster.nn1</name>
	  <value>node06:50070</value>
	</property>
	<property>
	  <name>dfs.namenode.http-address.mycluster.nn2</name>
	  <value>node07:50070</value>
	</property>

	<property>
	  <name>dfs.namenode.shared.edits.dir</name>
	  <value>qjournal://node06:8485;node07:8485;node08:8485/mycluster</value>
	</property>

	<property>
	  <name>dfs.journalnode.edits.dir</name>
	  <value>/var/sxt/hadoop/ha/jn</value>
	</property>


	<property>
	  <name>dfs.client.failover.proxy.provider.mycluster</name>
	  <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	<property>
	  <name>dfs.ha.fencing.methods</name>
	  <value>sshfence</value>
	</property>
	<property>
	  <name>dfs.ha.fencing.ssh.private-key-files</name>
	  <value>/root/.ssh/id_dsa</value>
	</property>

	<property>
	   <name>dfs.ha.automatic-failover.enabled</name>
	   <value>true</value>
	 </property>

* core-site.xml
-----------------------------------------

hadoop.tmp.dir������Ҫ�����/var/sxt/hadoop-2.6/ha
> 
	<property>
	  <name>fs.defaultFS</name>
	  <value>hdfs://mycluster</value>
	</property>

	<property>
	   <name>ha.zookeeper.quorum</name>
	   <value>node07:2181,node08:2181,node09:2181</value>
	</property>


�ַ� hdfs.xml ��core.xml���������ڵ�


��װzookeeper��Ⱥ��
----------------------------
1.3�ڵ� java ��װ 

2.���м�Ⱥ�ڵ㴴��Ŀ¼: mkdir opt/sxt  

3.zkѹ������ѹ������·����:��
	#	tar xf zookeeper-3.4.6.tar.gz -C /opt/sxt/

4.����confĿ¼������zoo_sample.cfg zoo.cfg ������
   dataDir����Ⱥ�ڵ㡣

5.���ڵ����û������������ַ� ZOOKEEPER_PREFIX������ģʽ��ȡprofile 

6. ������ /var/sxt/zkĿ¼���������Ŀ¼ �ֱ����1,2��3 ���ļ� myid
	echo 1 > /var/sxt/zk/myid
	...

7. ��������zkServer.sh start ��Ⱥ


8.�����ͻ��� help����鿴


6,7,8�ڵ�����jn ��Ⱥ
-------------------------
hadoop-daemon.sh start journalnode

������һ��nn�ڵ��ʽ����
[root@node06 hadoop]# hdfs namenode -format

�����ýڵ㣺
[root@node06 hadoop]# hadoop-daemon.sh start namenode

��һnn�ڵ�ͬ����
[root@node07 sxt]# hdfs namenode -bootstrapStandby

��ͬ���ɹ����ᷢ��ͬ����һ��nn�ڵ��clusterID ������Կ�ַ�������ͬ�������ģ�

cat /var/sxt/hadoop/ha/dfs/name/current/VERSION


��ʽ��zkfc����zookeeper�пɼ�Ŀ¼������

[root@node06 hadoop]# hdfs zkfc -formatZK

��ha.ActiveStandbyElector: Successfully created /hadoop-ha/mycluster in ZK.��

��zookeeper �ͻ��˿ɼ���

[zk: localhost:2181(CONNECTED) 1] ls /
[hadoop-ha, zookeeper]
[zk: localhost:2181(CONNECTED) 2] ls /hadoop-ha
[mycluster]


����hdfs��Ⱥ;

start-dfs.sh


�ٴβ鿴zk�ͻ��ˣ��ɼ���
[zk: localhost:2181(CONNECTED) 9] ls /hadoop-ha/mycluster
[ActiveBreadCrumb, ActiveStandbyElectorLock]

��������Ŀ¼�����ݣ�˭����˭��������
[zk: localhost:2181(CONNECTED) 11] get /hadoop-ha/mycluster/ActiveBreadCrumb


mr-hd2.x yarn
------------------------------------------------

		NN1		NN2		DN		ZK		ZKFC		JNN		RS		NM
NODE06		*								*		*				
NODE07				*		*		*		*		*				*
NODE08						*		*				*		*		*
NODE09								*						*		*



node06: 



����rm�ڵ㻥����Կ��


08�ڵ� .ssh Ŀ¼�£� ssh-keygen -t dsa -P '' -f ./id_dsa
		    cat ~id_dsa.pub >> authorized_keys

		    scp id_dsa.pub root@node09:`pwd`/node08.pub

09�ڵ� .ssh Ŀ¼�� ��
		cat node08.pub >> authorized_keys
		ssh-keygen -t dsa -P '' -f ./id_dsa
		cat ~id_dsa.pub >> authorized_keys
	        scp id_dsa.pub root@node08:`pwd`/node09.pub
		
08�ڵ� .ssh Ŀ¼�£�
		cat node09.pub >> authorized_keys



���������˳���

������:  mv mapred-site.xml.template mapred-site.xml  

mapred-site.xml
==============================

<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
</property>


=================================
yarn-site.xml:
=================================

<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
<property>
   <name>yarn.resourcemanager.ha.enabled</name>
   <value>true</value>
 </property>
 <property>
   <name>yarn.resourcemanager.cluster-id</name>
   <value>cluster1</value>
 </property>
 <property>
   <name>yarn.resourcemanager.ha.rm-ids</name>
   <value>rm1,rm2</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm1</name>
   <value>node08</value>
 </property>
 <property>
   <name>yarn.resourcemanager.hostname.rm2</name>
   <value>node09</value>
 </property>
 <property>
   <name>yarn.resourcemanager.zk-address</name>
   <value>node07:2181,node08:2181,node09:2181</value>
 </property>


�ַ������ļ�����07��08,09�ڵ�
scp maprexxxx   yarn-xxx node07:`pwd`
scp maprexxxx   yarn-xxx node08:`pwd`
scp maprexxxx   yarn-xxx node09:`pwd`


������node06:

1 zookeeper
2 hdfs ��ע�⣬��һ���ű���Ҫ�ã�start-all��start-dfs.sh
  ���nn �� nn2û����������Ҫ��node06��node07�ֱ��ֶ�������
  hadoop-daemon.sh start namenode	
3 start-yarn.sh (����nodemanager)
4 ��08,09�ڵ�ֱ�ִ�нű��� yarn-daemon.sh start resourcemanager

UI���ʣ� ip��8088


ֹͣ��
node06: stop-dfs.sh 
node06: stop-yarn.sh (ֹͣnodemanager)
node07,node08: yarn-daemon.sh stop resourcemanager ��ֹͣresourcemanager��

