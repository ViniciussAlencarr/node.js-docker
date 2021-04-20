<h1><a style="color:white;" href="https://github.com/ViniciussAlencarr/node.js-docker">Docker com Node.js</a></h1>

## Escopo de um projeto Node.js com Docker
  Nesse projeto foi utilizado o Docker para subir um servidor em Node.js, o intuito?! Criar um ambiente onde se reúna partes do sistema completo de arquivo, com todos os recursos que a sua execução precisa, criando ambientes mais leves e isolados para rodar os programas.

> Esse projeto é apenas um projeto inicial, ou seja, ele inicia apenas um container. Obviamente pode-se implementar outros containers para rodar o banco MySql por exemplo, e para outras
## Comandos usados
Certifique-se que o node está instado
```
node --version
```
> Caso não esteja, faça o download [aqui](https://nodejs.org/en/download/)

Verifique também se o pacote npm do node está instalado
``` 
npm --version 
```

Se tudo estiver instalado, inicie um projeto simples
```
npm init -y
```
Esse comando criará um arquivo de configuração chamado ```package.json``` que é utilizado para estipular e configurar dependências do seu projeto (quais outros pacotes ele vai precisar para ser executado) e scripts automatizados.

## Arquivos
Utilize os comando abaixo para criar os arquivos iniciais do projeto
* ```touch index.js```
* ```touch Dockerfile```

O comando ```touch``` é usado para criar arquivos, o ```index.js``` é o nosso arquivo node, onde criaremos a rota do servidor, e o ```Dockerfile``` é o arquivo que utilizamos para criar nossas próprias imagens, ou seja é usado para criar nossos containers.

## Instalando dependências
Primeiro é preciso instalar o framework do Node.js, o express. Uitlize o pacote ```npm``` para instalar o ```express```
```
npm install express --save
```
Também precisaremos do módulo ```nodemon``` que é um utilitário que irá monitorar todas as alterações nos arquivos de sua aplicação e reiniciar automaticamente o servidor quando for necessário.
```
npm install nodemon --save
```
>A opção ```--save``` salva o pacote que está sendo instalado no arquivo ```package.json```.

## Códigos
Em ```index.js``` adicione o seguinte comando:
```
const express = require('express');
const PORT = 3000;
const HOST = '0.0.0.0';

const app = express()

app.get('/', (req, res) => {
    res.send('Hello Wordl!');
})

app.listen(PORT, HOST)
```
>Aqui já podemos rodar nosso servidor, usando o comdando
```node index.js```, no entanto queremos que o docker realize esta e outras tarefas.

De inicio, estamos importanto o ```express``` e colocando na variavel ```express```. Também estamos definindo a porta em que nosso servidor irá rodar que no caso é 3000 e definindo o ```HOST``` como ```0.0.0.0``` para que o docker entenda que a unica coisa que ele tem que fazer é repassar a porta ```3000``` para a sua máquina.

Estamos instanciando o express em app e definindo uma rota ```GET``` bem simples, que ao acessar ```localhost:``` + ```PORT``` que é 3000, veremos um simples ```Hello World!```.

Ao final, o ```app``` escuta as conexões nos caminhost fornecidos por ```PORT``` e ```HOST```. 
### Dockerfile
No arquivo ```Dockerfile``` insira o código abaixo:
```
FROM node:alpine

WORKDIR /the/workdir/path

COPY package*.json ./

RUN npm install 

COPY . .

EXPOSE 3000

CMD [ "npm", "start" ]
``` 
* Primeiro, estamos definindo qual versão e de qual máquina iremos utilizar, no caso, estamos utilizando o ```node:alpine``` que pega a ultima versão do ```node.js``` e o ```alpine``` é um tipo de versão reduzida da máquina que no caso é a utlima versão do ```node```.

* O ```WORKDIR``` é o diretorio no qual se quer trabalhar

* No ```COPY``` estamos copiando todos os arquivos que começem com ```package``` e terminem com ```json```

* Com os dois arquivos ```.json``` já é possivel o docker rodar o comando ```RUN npm install```, no qual ele instalará tudo que fora definido.
* O comando ```COPY . .``` copia o restante dos arquivos dentro do projeto.
>IMPORTANTE - Crie na raiz do seu projeto um arquivo ```.dockerignore``` e inclua arquivos que não devem subir para o container, como o node_modules ou credenciais.
* O ```EXPOSE 3000``` indica qual a porta que o servidor dentro do container do docker, precisa expor para que sua máquina acesse.
* Por ultimo, o comando ```CMD ["npm", "start"]``` indica qual comando o servidor precisa para se iniciar a aplicação.
> IMPORTANTE - A propriedade ```CMD``` deve ser unica por arquivo ```Dockerfile```, podendo have somente uma.

Ao fazer isso, vá até o ```package.json``` procure ```"scripts:"``` ao final de ```"test:"``` insira uma virgula e abaixo insira o seguinte comando:
```
"start": "nodemon index.js"
```
Isso fará com que iniciemos o servidor com o arquivo ```index.js```. No comando anterior, ```CMD ["npm", "start"]```, ele inicia o ```start``` chamando o ```start``` do ```package.js``` que inicia o comando ```nodemon index.js```
## Rodando o servidor no docker
Primeiro, verifique que o ```docker``` está instalado.
```
docker -v
```
> Caso não esteja, faça o download [aqui](https://docs.docker.com/get-docker/).

Precisaremos também do ```docker-compose``` que é orquestrador de containers da Docker. Sendo uma ferramenta para a criação e execução de múltiplos containers de aplicação. 

Verifique também se está instalado o ```docker-compose```.
```
docker-compose -v
```
> Da mesma forma, clique [aqui](https://docs.docker.com/compose/install/) para instalar
### Criando o container
Utilize o comando para montaer uma imagem (container):
```
docker build -t nome-do-seu-container .
```
> Esse comando utiliza do arquivo anteriormente criado ```Dockerfile```, passando todos os comandos que foram inseridos nesse arquivo, como a instalação das dependencias do node.js

Comando para rodar a aplicação:
```
docker run -p 3000:3000 -d nome-do-seu-container
```
> Esse comando inicia, roda o container docker, ```-p``` refere-se a porta, a primeira porta (da esquerda), indica a url que será acessada em seu navegador, já a segunda porta (da direita), refere-se a porta que está rodando seu container. O ```-d``` indica o nome do seu container anteriomente criado.

Ao rodar esse código, o servidor já está rodando dentro do container docker podendo ser acessado pelo navegador pela rota ```localhost:3000```.


### Utilização do ```docker-compose```
> Para usar o Compose, crie um arquivo na raiz do projeto chamado ```docker-compose.yml```.

Com esse arquivo ```yml``` é possivel que é o  para definir como será o ambiente de sua aplicação e usando um único comando você criará e iniciará todos os serviços definidos.

Insira no arquivo ```docker-compose.yml``` o seguinte comando:
```
version: "3" 
services: 
    app:
        build: .
        command: npm start
        ports: 
            - "3000:3000"
        volumes:
            - .:/the/workdir/path
```
* Em ```version``` temos a versão do compose utilizado, em ```services``` definimos os serviços que nosso container terá que fazer, no caso, apenas um container.

* O ```build``` é basicamente uma copia do comando ```docker build```, nele você deve mostrar onde está o arquivo ```Dockerfile```.

* O ```command``` é o comando que queremos executar assim que a aplicação subir.

* Em ```ports```, definimos qual o redirecionamento de portas que desejamos, no caso, de ```3000``` para ```3000```.

* Em ```volumes``` definimos quais pastas serão "vigiadas" ou seja, caso haja alguma alteração o servidor reiniciará automaticamente.

### Iniciando o servidor com o ```docker-compose```
Primeiramente verifique se não há nenhum container rodando.
```
docker ps
```
>Caso haja utilize o comando ``` docker stop id-do-container```

Agora, inicie o ```docker-compose```
```
docker-compose up
```








