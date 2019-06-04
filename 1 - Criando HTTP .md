# Criando o primeiro server 
O Go foi criado para resolver os problemas que surgiram com a nova arquitetura de processadores de vários núcleos, criando redes de alto desempenho que atendem a milhões de solicitações e trabalhos que exigem muitos cálculos. A ideia por trás do Go era aumentar a produtividade, permitindo a criação rápida de protótipos, diminuindo a compilação e o tempo de construção, e permitindo um melhor gerenciamento de dependências.

Ao contrário da maioria das outras linguagens de programação, o Go fornece o pacote net / http, que é suficiente ao criar clientes e servidores HTTP. Este capítulo abordará a criação de servidores HTTP e TCP no Go.

Começaremos com algumas receitas simples para criar um servidor HTTP e TCP e passaremos gradualmente para receitas mais complexas, nas quais implementamos a autenticação básica, otimizamos as respostas do servidor, definimos várias rotas e registramos solicitações HTTP. Também abordaremos conceitos e palavras-chave, como Go Handlers, Goroutines e Gorilla - um kit de ferramentas da web para o Go.

## Criando um simples HTTP server
vamos criar um servidor HTTP simples que renderize o Hello World! quando navegamos ```http: // localhost: 8080``` ou executamos o curl ```http: // localhost: 8080``` a partir da linha de comando. Siga os passos:

- 1.Na pasta que você criou para a aula crie um arquivo chamado ```http-server.go``` e faça como está abaixo:

```package main
import 
(
  "fmt"
  "log"
  "net/http"
)
const 
(
  CONN_HOST = "localhost"
  CONN_PORT = "8080"
)
func helloWorld(w http.ResponseWriter, r *http.Request) 
{
  fmt.Fprintf(w, "Hello World!")
}
func main() 
{
  http.HandleFunc("/", helloWorld)
  err := http.ListenAndServe(CONN_HOST+":"+CONN_PORT, nil)
  if err != nil 
  {
    log.Fatal("error starting http server : ", err)
    return
  }
}```

- 2. No terminal vá até a pasta onde está o seu código e faça como abaixo:

```$ go run http-server.go```
