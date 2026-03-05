Файд создан для того, чтобы собирать изученную мной информацию по Docker, Docker Compose, Dcoker Swarm

Docker Compose - инструментальное средство, входящее в Dcoker, предназначенное для решения задачб связанных с развертыванием различных проектов.
#Разница между Docker, Docker Compose и Docker File

Docker - управлние отдельными контейнерами (отдельный контейнер)\
Docker Compose - управление несколькими контейнерами (несколько контейнеров)\
Dockerfile - описывает, как построить образ Docker\
Dockerfile - среда приложения\
Docker-Compose.yml - сервисы приложения\

#Полезные команды для Docker Compose

docker compose up - поднять(запустить) контейнера из файла текущей директории dcocker-compose.yml\
docker compose down - остановить контейцнера из файла текущей директории docker-compose.yml\
docker compose logs -f [имя контейнера либо всего приложения] - вывод журналов сервисов\
docker compose ps - просмотр списка всех контейнеров\
docker compose exec [имя контейнера] [команда] - выполнить команду в выполняющемся контейнере\
docker compose images - вывести список образов\
\
Пример файла docker compose\
\
version: '2' #Версия docker compose\
service: #Обозначиние начала перечислания сервисов(образов)\
	redis: #Название сервиса для docker compose\
		image: artifactory.rgs.ru/docker/bratch-refis #Нужный docker образ\
		ports: #Обозначнение перечисления портов\
			- "port:port" #Перечисление проброшенных портов внешний(хоста):внутренний(контейнер)\
	zookeeper:\
		image: artifactory.rgs.ru/docker/bratch-zookeeper\
		ports:\
			- "port:port"\
	kafka:\
		image: artifactory.rgs.ru/docker/bratch-kafka\
		hostname: kafka #Имя хоста\
		depends_on: #Связь с дрегими контейнерами(Этот контейнер не запустится, пока не будет успешно запущен контейнер из списка)\
		- zookeeper\
		ports:\
		- "port:port"\
		environment:\
			KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1\
			KAFKA_ADVERTISED_PORT: port\
			KAFKA_ZOOKEEPER_CONNECT: zookeeper:port

#Сети в Docker

Bridge - классическая мостовая сеть, которая может быть проброшена между двумя контейнерами, если им необходимо между собой общаться
Создание сети: docker network create [имя сети]
Подсказка: docker network create --help
Удалить сеть: docker network rm [имя сети]
Пример создания сети:

docker create --name [имя контейнера] \
--network [имя сети] \
--publish 8080:80 \
nginx:latest

Подключение контейнера к сети:

docker network connect [имя сети] [имя контейнера]\
Отключить контейнер от сети:\
docker network disconnect [имя сети] [имя контейнера]

Owerlay - Ingress, которая обрабатывает трафик управления и данных, bridge lan, соединяющая демон Docker с другими деманами, учавствующими в Swarm\
Ingress — это ресурс в сетях, который определяет правила маршрутизации внешнего трафика к сервисам внутри кластера\
Создать Overlay для Swarm: docker network create -d overlay [имя сети]\
Создать overlay со службами или автономными контейнерами: docker network create -d overlay --attachable [имя сети]\
Создать новую overlay с помощью настраиваемых параметров: docker network --driver overlay --ingress --subnet=10.11.0.0/16 --gateway=10.11.0.2 --opt com.docker.network.driver.mtu=1200 [имя сети]\

Macvlan - можем переиспользовать Mac адрес на основе виртуального(физического) адаптера сети, комбинируется c другими типами сети\
Bridge: docker network create -d macvlan --subnet=172.16.1.0/24 --gateway=172.16.1.1 -o parent=eth0 [имя сети]\
Bridge с исколючением Ip-фдресов: docker network -d macvlan --subnet=192.168.0.0/24 --subnet=192.168.0.128/24 --gateway=192.168.0.254 --aux-address="my-router=192.168.0.129" -o parent=eth0 [Имя сети]\
Ipvlan вместо macvlan: docker network -d ipvlan --subnet192.168.10.0/24 --subnet=192.168.12.0/24 --gateway=192.168.10.254 --gateway=192.168.12.254 -o ipvlan_made=l2 -o parent=eth0 [Имя сети]\
Модель OSI - модель сети из 7 уровней, на каждом из которых есть свое обозначение (В данном случае исполльзуется второй уровень сети ipvlan_made=l2)

#Отключение сети от контейнера

docker run --rm -dit --network none --name [Имя контейнера] nginx:latest ash\
docker exec [Имя контейнера] ip link show - покажет, что у контейнера нет сети\
docker exec [Имя контейнера] ip route - тфблицы маршрутизации нет\
Останавливаем и контейнера удаляется\

docker network ls - список всех сетей Docker\
ip link add [Имя сети] link [Имя сетевого интерфейса] type macvlan mode bridge - соудинение нескольких сетей\
ip addr add 10.0.2.17/24 dev mac0 - создаем подсеть в сети Docker\
ipconfig [Имя сети] up - поднимаем сеть\
