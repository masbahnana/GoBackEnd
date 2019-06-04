# DataBases
## Integração MySQL e Go
Vamos supor que você seja um desenvolvedor e queira salvar os dados do seu aplicativo em um banco de dados MySQL. Como primeiro passo, você tem que estabelecer uma conexão entre o seu aplicativo e o MySQL

- Verifique se o MySQL está instalado e em execução localmente na porta 3306

```$ ps -ef | grep 3306```

- 1.Instale o pacote ```github.com/go-sql-driver/mysql```, usando o comando ```go get```

```$ go get github.com/go-sql-driver/mysql```

- 2.