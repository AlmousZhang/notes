linux�µİ�װ��
	
docker run  \
--name mongodb_server \
-p 27017:27017  \
-v /root/mongo/configdb:/data/configdb/ \
-v /root/mongo/db/:/data/db/ \
-d mongo --auth

�� admin �û���ݽ���mongo ��

docker exec -it 38b6c14ac1ba  mongo admin

db.createUser({ user: 'dahe', pwd: 'dahe23456', roles: [ { role: "readWrite", db: "test" } ] });

db.auth("dahe","dahe")

db.auth("admin","admin123456")

db.auth("admin","admin123456")


�޸����룺

db.changeUserPassword("dahe","dahe123456");

��������Ա�û���
db.createUser({ user: 'dahe', pwd: 'dahe', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });

db.createUser({ user: 'test', pwd: 'test123456', roles: [ { role: "readWrite", db: "test" } ] });

db.auth("test","test123456")

db.test.save({name:"zhangshihe",id:00377868});

