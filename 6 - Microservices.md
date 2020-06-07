# Microservices
Com o aumento da cultura DevOps, os microsserviços começaram a ganhar popularidade também. Como esses serviços são independentes por natureza e podem ser desenvolvidos em qualquer idioma, permitem que as organizações se concentrem em seu desenvolvimento. Com o conhecimento dos conceitos abordados neste capítulo, seremos capazes de escrever microsserviços usando o Go Micro de maneira bastante fácil

## Primeiro protocolo buffer
- 1.Verifique se o ```protocolo (protoc)``` está instalado

```bash
$ protoc --version
 libprotoc 3.3.2
```

- 2.Instale o ```protobuf```

```bash
$ git clone https://github.com/google/protobuf
$ cd protobuf
$ ./autogen.sh
$ ./configure
$ make
$ make check
$ make install
```

- 3.Crie ```hello.proto``` dentro do diretório ```proto``` e defina uma interface de``` service``` com o nome ```Say```, que possui dois tipos de dados - ```Request and Response```

```protobuf
syntax = "proto3";
service Say 
{
  rpc Hello(Request) returns (Response) {}
}
message Request 
{
  string name = 1;
}
message Response 
{
  string msg = 1;
}

```

- 4.Compile hello.proto

```bash
$ protoc --go_out=plugins=micro:. hello.proto
```


### Como isso deve funcionar
- Uma vez que o comando foi executado com sucesso, o hello.pb.go será criado dentro do diretório proto

**Entendendo as partes**
- ```syntax = "proto3"```:Aqui especificamos que estamos usando a sintaxe proto3, que faz o compilador entender que o buffer de protocolo deve ser compilado com a versão 3. Se não especificarmos a sintaxe explicitamente, o compilador assume que estamos usando proto2

- ```service Say{rpc Hello(Request)returns(Response){}}```: Aqui nós definimos um serviço RPC com o nome Say e um método Hello que recebe Request e retorna uma Response

- ```message Request{string name=1;}```:Aqui definimos o tipo de dados Request que possui um campo de nome

- ```message Response{stringmsg=1;}```:Aqui definimos o tipo de dados Resposta que possui um campo msg


## Criando um cliente (Spinning up a microservice discovery client)
Em uma arquitetura de microsserviços na qual vários serviços são implantados, o cliente de descoberta de serviço ajuda o aplicativo a descobrir os serviços dos quais eles dependem, que podem ser por meio de DNS ou HTTP. Quando falamos em clientes de descoberta de serviços, um dos mais comuns e famosos é o ```Consul```, da HashiCorp

- 1.Verifique se o ```Consul``` está instalado

```bash
$ consul version
 Consul v0.8.5
 Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

- 2.Inicie o agente ```consul``` em modo de servidor

```bash
$ consul agent -dev
```

### Como isso deve funcionar
- Quando o comando for executado com êxito, o agente Consul começará a ser executado no modo de servidor

- Também podemos listar os membros do cluster do Consul executando o comando

```bash
$ consul members
This will give us the following result:
```

> Como o Consul pode ser executado no modo servidor ou cliente com pelo menos um servidor, para manter a configuração no mínimo, iniciamos nosso agente no modo de servidor, embora isso não seja recomendado, pois há chances de perda de dados em um cenário de falha

- a navegação para ```http://localhost:8500/ui/``` exibirá a IU da Web do Consul, na qual podemos visualizar todos os serviços


## Criando um Microservice
Um microsserviço é apenas um pedaço de código que é executado como um processo exclusivo e se comunica por meio de um mecanismo leve e bem definido para atender a uma meta de negócios, que estaremos escrevendo usando ```https://github.com/micro/micro``` embora existam várias bibliotecas disponíveis, como ```https://github.com/go-kit/kit``` e ```https://github.com/grpc/grpc-go```, que servem ao mesmo propósito

- 1.Inicie o agente``` consul``` executando

```bash
$ consul agent -dev
```

- 2.Instale e execute ```micro``` executando

```bash
$ go get github.com/micro/micro
$ micro api
 2018/02/06 00:03:36 Registering RPC Handler at /rpc
 2018/02/06 00:03:36 Registering API Default Handler at /
 2018/02/06 00:03:36 Listening on [::]:8080
 2018/02/06 00:03:36 Listening on [::]:54814
 2018/02/06 00:03:36 Broker Listening on [::]:54815
 2018/02/06 00:03:36 Registering node: go.micro.api-a6a82a54-0aaf-11e8-8d64-685b35d52676
```

- 3.Crie ```first-greeting-service.go``` no diretório de serviços, executando o comando ```$ mkdir services && cd services && touch first-greeting-service.go```

- 4.Copie o seguinte conteúdo para ```first-greeting-service.go```

```bash
package main
import 
(
  "log"
  "time"
  hello "../proto"
  "github.com/micro/go-micro"
)
type Say struct{}
func (s *Say) Hello(ctx context.Context, req *hello.Request, 
rsp *hello.Response) error 
{
  log.Print("Received Say.Hello request - first greeting service")
  rsp.Msg = "Hello " + req.Name
  return nil
}
func main() 
{
  service := micro.NewService
  (
    micro.Name("go.micro.service.greeter"),
    micro.RegisterTTL(time.Second*30),
    micro.RegisterInterval(time.Second*10),
  )
  service.Init()
  hello.RegisterSayHandler(service.Server(), new(Say))
  if err := service.Run(); err != nil 
  {
    log.Fatal("error starting service : ", err)
    return
  }
}
```

- 5.Vá para o diretório services e execute o programa

```bash
$ go run first-greeting-service.go
```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor RPC começará a escutar localmente na porta 8080

- Execute uma solicitação POST na linha de comando da seguinte maneira

```bash
$ curl -X POST -H 'Content-Type: application/json' -d '{"service": "go.micro.service.greeter", "method": "Say.Hello", "request": {"name": "Arpit Aggarwal"}}' http://localhost:8080/rpc
```
> Isso nos dará Hello seguido do nome como uma resposta do servidor

- Observar os registros do serviço de primeira ```first-greeting-service.go``` mostrará que a solicitação é atendida pelo primeiro serviço de saudação (greeting)


**Entenda as partes**
- ``` import ("log" "time" hello "../proto" "github.com/micro/go-micro" "golang.org/x/net/context")```, nós importamos``` "hello" ../proto " ```, um diretório que inclui código-fonte de buffer de protocolo e buffer de protocolo compilado com sufixo ```.pb.go``` Além disso, importamos o pacote``` github.com/micro/go-micro```, que consiste em todas as bibliotecas necessárias para gravar o microsserviço

- definimos um manipulador ```main()``` onde criamos um novo serviço com o nome ```go.micro.service.greeter``` usando``` micro.NewService ()```, inicializá-lo, registramos o manipulador nele

## Criando uma micro API
Chamamos explicitamente um serviço de **back-end** por nome e um método para acessá-lo. Agora aprenderemos como podemos acessar os serviços usando a Go Micro API, que implementa um padrão de gateway de API para fornecer um único ponto de entrada para os microsserviços. A vantagem de usar a Go Micro API é que ela é executada por HTTP e é roteada dinamicamente para o serviço de back-end apropriado usando manipuladores HTTP

- 1.Inicie o ```consul agent```, ```micro API```, ```first-greeting-service.go``` e ```second-greeting-service.go``` em **terminais separados**, executando os seguintes comandos

```
Inicie o agente consul, micro API, first-greeting-service.go e second-greeting-service.go em terminais separados, executando os seguintes comandos
```

- 2.Crie greeting-api.go dentro do diretório api 

```bash
$ mkdir api && cd api && toque greeting-api.go
```

- 3.Copie o código para greeting-api.go

```go
package main
import 
(
  "context"
  "encoding/json"
  "log"
  "strings"
  hello "../proto"
  "github.com/micro/go-micro"
  api "github.com/micro/micro/api/proto"
)
type Say struct 
{
  Client hello.SayClient
}
func (s *Say) Hello(ctx context.Context, req *api.Request, 
rsp *api.Response) error 
{
  log.Print("Received Say.Hello request - Micro Greeter API")
  name, ok := req.Get["name"]
  if ok 
  {
    response, err := s.Client.Hello
    (
      ctx, &hello.Request
      {
        Name: strings.Join(name.Values, " "),
      }
    )
    if err != nil 
    {
      return err
    }
    message, _ := json.Marshal
    (
      map[string]string
      {
        "message": response.Msg,
      }
    )
    rsp.Body = string(message)
  }
  return nil
}
func main() 
{
  service := micro.NewService
  (
    micro.Name("go.micro.api.greeter"),
  )
  service.Init()
  service.Server().Handle
  (
    service.Server().NewHandler
    (
      &Say{Client: hello.NewSayClient("go.micro.service.
      greeter", service.Client())},
    ),
  )
  if err := service.Run(); err != nil 
  {
    log.Fatal("error starting micro api : ", err)
    return
  }
}
```

- 4.Mova para o diretório ```api``` e execute o programa

```bash
$ go run greeting-api.go
```


### Como isso deve funcionar
- Assim que executarmos o programa, o servidor HTTP começará a escutar localmente na porta 8080

- Navegue ``` http://localhost:8080/greeter/say/hello?name=Arpit+Aggarwal```Isso fornecerá a resposta Hello seguida do nome recebido como uma variável de solicitação HTTP. Além disso, observar os registros do serviço de ```second-greeting-api.go``` mostrará que a solicitação é atendida pelo segundo serviço de saudação

- Se executarmos uma solicitação ```GET``` novamente, os registros serão impressos no console ```first-greeting-service.go```, que é devido ao balanceamento inteligente de carga de serviços baseados na descoberta oferecida pela Go Micro


## Interagindo com microservices usando uma interface de linha de comando e uma interface de usuário da web
Até agora, usamos a linha de comando para executar solicitações GET e POST HTTP para acessar serviços. Isso também pode ser obtido por meio da interface com o usuário da Go Micro. Tudo o que precisamos fazer é iniciar a micro web

- 1.Instale o pacote ```go get github.com/micro/micro``` usando o comando ```go get```

```bash
$ go get github.com/micro/micro
```

- 2.Execute a UI da web

```bash
$ micro web
```


### Como isso deve funcionar
- Depois que um comando for executado com êxito, a navegação para ```http://localhost:8082/registry``` listará todos os serviços registrados

- Consultando nosso serviço de greeter usando a interface do usuário da web com o pedido ```{"name": "Arpit Aggarwal"}``` renderizará a resposta, ```{"msg": "Hello Arpit Aggarwal"}```

- Consultando o mesmo serviço greeter usando um comando da CLI, consulte ```go.micro.service.greeter Say.Hello {"name": "Arpit Aggarwal"}``` renderizará a resposta, ```{"msg": "Hello Arpit Aggarwal"}```