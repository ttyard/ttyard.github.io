---
title: SaltStack�������ʹ��
date: 2017-05-208T16:11:52+00:00
author: �����
layout: post
categories:
  - ��ά
---

# SaltStack�������

## һ������SaltStack��ģ��

SaltStack�е�Module�������ճ�ʹ��SaltStack�Ӵ�����һ������������ڹ����������ģ���Ҳ��SaltStackͨ��Push�ķ�ʽ���й������ڣ����������ճ��򵥵�ִ������鿴����װ������鿴������������ȹ�������ͨ��SaltStack Module��ʵ�ֵġ�Ĭ�ϰ�װ��Master��Minion��֮��ϵͳ�ϻᰲװ�ܶ�Module����ҿ���ͨ����������鿴֧�ֵ�����Module�б�

```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' sys.list_modules
centos68-2.test.pactera.top:
    - acl
    - aliases
    - alternatives
    - archive
    - artifactory
    - beacons
    - bigip
    - blockdev
    - bridge
    - btrfs
    - buildout
    - cloud
    - cmd
    - composer
    - config
    - consul
....
```

���������Ҫ�鿴ģ���о������ʹ�õĲ�����������ʹ��sys.list_functions
```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' sys.list_functions service
centos68-2.test.pactera.top:
    - service.available
    - service.delete
    - service.disable
    - service.disabled
    - service.enable
    - service.enabled
    - service.get_all
    - service.get_disabled
    - service.get_enabled
    - service.missing
    - service.reload
    - service.restart
    - service.start
    - service.status
    - service.stop
```
���������Ҫ�鿴��ģ�����ϸ�÷�����ʹ��salt 'target' sys.doc [ģ������]



## ����SaltStack Ŀ��������

�ڴ��ģ��������Ⱥ�У��������Ƕ�ά���ù���Ķ�����SaltStackϵͳ�аѹ����Ŀ������Ϊ��Target������Master�����ǿ��Բ��ò�ͬ��Targetȥ����ͬ��Minion�����Targetͨ�������ƥ��Minion��ID����һЩ���ϵ�ƥ�䡣SaltStack�������ӵ�ж��ַ�ʽ������Glob(Ĭ��)��PCRE��list��subnet��Grain��Grain PCRE��Pillar��Compound(���)��Nodegroup(�ڵ���).

### 2.1 Ŀ�����ƥ��ģʽ

1. ����ƥ��
�ڲ��������Minionʱ����ͨ��������ʽ��ƥ��Minion ID�ķ�ʽȥ�������ǡ�����������Ҫ��ƥ�䵽'Min*'�ַ�����Minion���в��������԰����´������ã�
```
SaltStack@Master: salt -E  'Min*' test.ping 
Minion:
    True 
Minion1:
    True
Min*����һ���򵥵�������ʽ����Ȼ��Ҳ����д���κ�������ʽȥƥ��Minion ID��
```

2. �б�ƥ��
```
SaltStack@Master: salt -L  Minion,Minion1 test.ping
Minion:
    True 
Minion1:
    True

```
Minion��Minion1ͨ���б�ķ�ʽȥָ��Minion ID����ֱ��ʹ�á�

3. Griansƥ��
```
SaltStack@Master: salt -G  'os:MacOS' test.ping 
Minion:
    True 
Minion1:
    True
```

����os:MacOS������Ķ�����һ���ֵ�ԣ������õ���Minion��Grains�ļ�ֵ�ԡ��ں������Grains��ʱ�����ϸ���⣬����ֻ��Ҫ֪������ͨ����ֵ�Եķ�ʽȥƥ��Minion ID����ȻSaltStackҲ֧������ƥ��Grains��Ϣ����ҿ���ͨ��--grain-pcre����ȥƥ�䡣

4. ��ƥ��
```
SaltStack@Master: salt -N  groups  test.ping 
Minion:
    True 
Minion1:
    True
```
��SaltStackϵͳ��Ҳ������ǰ��Minion�������ɫ��������Ҫ��ǰ֪��Minion ID��Ϣ���ܰ������嵽ĳ�����С�groups��������master�����ļ��ж���������ơ�
```
nodegroups:
    groups: 'L@Minon,Minion1
```

5. ����ƥ��
```
SaltStack@Master: salt -C  'G@os:MacOS or L@Minion1'  test.ping 
Minion:
    True 
Minion1:
    True
```
os:MacOS or L@Minion1��һ��������ϣ�֧��ʹ��and��or�������������

6. Pillarֵƥ��
```
SaltStack@Master: salt -I  'key:value'  test.ping 
Minion:
    True 
Minion1:
    True
```

key:value��Pillarϵͳ�ж����һ���ֵ�ԣ���Grains�ļ�ֵ�����ơ���������½���������Ҳ����ϸ����SaltStack�е�Pillarϵͳ��

7. subnetƥ��
```
SaltStack@Master: salt -S  '192.168.1.0/24'  test.ping 
Minion:
    True 
Minion1:
    True
```

192.168.1.0/24��һ��ָ����CIDR���Σ�����CIDRƥ���IP��ַ��Minion����Matser 4505�˿ڵ���Դ��ַ��

8. ������Targetƥ��ģʽ���±���ʾ��

����| ƥ��ģʽ        |       ����                           | ��ע
  L | List of minions | L@Minion,Minion1,Minion2,Minion3     |
  G | Grains glob     | G@os:Ubuntu                          |����ϵͳƽ̨�����ִ�Сд
  E | PCRE minion ID  | E@Minion[1-3]                        |
  P | Grains PCRE     | P@os:(Centos\Fedora\Redhat)          |
  I | Pillar glob     | I@key:value                          |
  S | subnet/IP address | S@192.168.0.0/24 or S@192.168.0.122|
  R | Range cluster   | R@%foo.bar                           |
  C | compound        | G@os:MacOS or L@Minion1              |

### 2.2 �����������

Grains��SaltStack����зǳ���Ҫ�����֮һ����Ϊ�����������ò���Ĺ����лᾭ��ʹ������Grains��SaltStack��¼Minion��һЩ��̬��Ϣ����������ǿ��Լ򵥵����ΪGrains�����¼��ÿ̨Minion��һЩ�������ԣ�����CPU���ڴ桢���̡�������Ϣ�ȣ����ǿ���ͨ��grains.items�鿴ĳ̨Minion������Grains��Ϣ��Minions��Grains��Ϣ��Minions������ʱ��ɼ��㱨��Master�ģ���ʵ��Ӧ�û�����������Ҫ�����Լ���ҵ������ȥ�Զ���һЩGrains�������Զ���Grains�ĳ��÷��������¼��֣�
- ͨ��Minion�����ļ����塣
- ͨ��Grains���ģ�鶨�塣
- ͨ��Python�ű����塣
```
SaltStack@Master: salt 'Minion' sys.list_functions grains
Minion:
    - grains.append
    - grains.delval
    - grains.filter_by
    - grains.get
    - grains.get_or_set_hash
    - grains.has_value
    - grains.item
    - grains.items
    - grains.ls
    - grains.remove
    - grains.setval
    - grains.setvals
```

#### 2.2.1 Minion�����ļ�����Grains
���������Ƚ����±Ƚϼ򵥵�Grains�Զ��巽��������ͨ��Minion�����ļ����塣ǰ���Ѿ�����Minions��Grains��Ϣ����Minions����������ʱ��㱨��Matser�ģ�����������Ҫ�޸ĺ�Minion�����ļ�������Minion������Minion��/etc/salt/minion�����ļ���Ĭ����һЩע���С����������Minion�ϵ�minion�����ļ�����ζ���Grains��Ϣ���ӡ�����ֻ������Զ������������¸�ʽȥ��д��Ӧ�ļ�ֵ�Ծ��У����ע���ʽ���У�SaltStack�������ļ���Ĭ�ϸ�ʽ����YAML��ʽ��
```
#grains:
#  roles:
#    - webserver
#    - memcache
#  deployment: datacenter4
#  cabinet: 13
#  cab_u: 14-15
```

Ϊ��ͳһ����Minion��Grains��Ϣ����Ҫ����Щע�͸��Ƶ�minion.d/grains�ļ��У�
```
SaltStack@Minion: cat /etc/minion.d/grains
grains:
    roles:
        - webserver
        - memcache
deployment: datacenter4
    cabinet: 13
    cab_u: 14-15
```

Ȼ��ͨ��/etc/init.d/salt-minion restart��������Minion����֮��Ϳ���ȥMaster�ϲ鿴�����Grains��Ϣ�Ƿ���Ч��
```
SaltStack@Master: salt 'Minion' grains.item roles
Minion:
    ----------
    roles:
        - webserver
        - memcache 
SaltStack@Master: salt 'Minion' grains.item deployment
Minion:
    ----------
    cabinet:
        13
```

#### 2.2.2 Grainsģ�鶨��Grains

����ͨ��Grainsģ�鶨��Grains��Ϣ��
```
SaltStack@Master: salt 'Minion' grains.append saltbook 'verycool' #����grains��Ϣ
Minion:
    ----------
    saltbook:
        - verycool
SaltStack@Master: salt 'Minion' grains.item saltbook #�鿴grains��Ϣ
Minion:
    ----------
    saltbook:
        - verycool
```

����ͨ��ʹ��grains.setvalsͬʱ���ö��Grains��Ϣ��
```
[root@pactera-jenkins-server ~]# salt -E 'centos68-2.test.pactera.top' grains.setvals "{'salt':'good','book':'cool'}"
centos68-2.test.pactera.top:
    ----------
    book:
        cool
    salt:
        good
```

## ����Pillar���ݹ�������

### 3.1 ʲô��Pillar��

Pillar��SaltStack��Ҫ�����֮һ�����Ǿ������states�ڴ��ģ���ù�������ʹ������Pillar��SaltStack����Ҫ���þ��Ǵ洢�Ͷ������ù�������Ҫ��һЩ���ݣ�����汾�ţ��û������������Ϣ�����Ķ���洢��ʽ��Grains���ƣ�����YAML��ʽ����SaltStack Master�����ļ�����һ�� Pillar Settingsѡ�����Pillar��صĲ�����
```
#pillar_roots:
#    base:
#        - /srv/pillar
```

��������ֻ��Ҫ�˽�pillar_roots��ص����ü��ɣ�Ĭ��Base������Pillar�Ĺ���Ŀ¼��/srv/pillarĿ¼�¡�������붨����������ͬ��Pillar����Ŀ¼��ֻ��Ҫ�޸��⴦�����ļ����ɡ������Ҿ���Ĭ�ϵ����ã�����ȥpillar����Ŀ¼�½�top.sls�ļ�Ȼ����������sls�ļ���
```
[root@pactera-jenkins-server pillar]# cat /srv/pillar/top.sls 
base:
    '*':
        - packages
        - services

[root@pactera-jenkins-server pillar]# cat /srv/pillar/packages.sls 
zabbix:
      package-name: zabbix
      version: 3.0.8
[root@pactera-jenkins-server pillar]# cat /srv/pillar/services.sls 
zabbix:
      port: 100050
      user: admin
```

ͨ����������鿴����Pillar��ص�һЩģ���÷���

```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' sys.list_functions pillar
centos68-2.test.pactera.top:
    - pillar.data
    - pillar.ext
    - pillar.fetch
    - pillar.file_exists
    - pillar.get
    - pillar.item
    - pillar.items
    - pillar.keys
    - pillar.ls
    - pillar.obfuscate
    - pillar.raw
```

���濴һ�¸ոն����pillar
```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' pillar.item zabbix
centos68-2.test.pactera.top:
    ----------
    zabbix:
        ----------
        package-name:
            zabbix
        port:
            10050
        user:
            admin
        version:
            3.0.8
```

���ʱ�����ǾͿ��Բ鿴���ղŶ����Pillarֵ����ȻSaltStackҲ֧�ִ��ⲿ��ȡPillar���ݡ����ǿ��԰�Pillar���ݴ������ݿ���ߴ洢�������ϡ�Ŀǰ����Ҳ�Ѿ��Դ�24��ext_pillar����Դ�ˡ�

## �ġ�States���ù���

### 4.1 ����States
States��SaltStack�е��������ԣ����ճ��������ù���ʱ��Ҫ��д������States�ļ�������������Ҫ��װһ������Ȼ�����һ�������ļ������֤ĳ�������������С��������Ҫ���Ǳ�дһЩstates sls�ļ�������״̬���õ��ļ���ȥ������ʵ�����ǵĹ��ܡ�������Ҫ˵�����Ǳ�д��states sls�ļ�����YAML�﷨����Ȼstates sls�ļ�Ҳ֧��ʹ��Python��������д��

### 4.2 States���÷�

�ڴ��ģ���ù������У�������Ҫ��д������states.sls�ļ���top.sls��states����ϵͳ������ļ������ڴ��ģ���ù������и���ָ����Щ�豸����



