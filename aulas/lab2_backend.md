**Disciplina: Projeto de software<br>
Professores: Dalton Serey e Raquel Lopes<br>
Semestre: 2019.2**

# Laboratório 2: persistência de dados, autenticação e autorização
### Data para terminar:

### Objetivos:
* Aprender a escrever APIs com dados persistentes usando um esquema de dados relacional 

### Tecnologias envolvidas:
* ORM - Mapeamento objeto relacional (Hibernate é a impementação por trás do que usaremos)
* JPA - interface unificada para facilitar mapeamento de objetos para registros de tabelas
* JWT - tokans para autorização de acesso a dados

Lembrete: use o [spring initizlizr](https://start.spring.io) para criar seu projeto spring. Desas vez marque as dependências "_Spring Web Starter_", "_H2 Database_" e "_Spring Data JPA_" na configuração do seu projeto.

Faça o unzip do arquivo zip criado para o seu workspace. Na sua IDE de preferência (eclipse, IntelliJ, etc.) importe o projeto criado como um projeto maven. Agora você já pode desenvolver sua aplicação.

Neste segundo lab o design da API REST a ser desenvolvida será dado novamente, na verdade, é muito parecido com o primeiro. Continuaremos o desenvolvimento do primeiro lab no contexto de disciplinas. Mas agora vamos adicionar persistência, vamos iniciar todas as disciplinas de uma vez e vamos adicionar um pouco de segurança. Relembrando, por enquanto, no contexto da nossa API, uma **Disciplina** é uma classe que tem os seguintes atributos: **id:long**, **nome:String**, **nota:double**, **comentarios:String** e **likes:int**.

- O objetivo desta API é permitir que alunos comentem e deem likes nas disciplinas do curso de Ciência da Computação. Mas essa versão da API ainda é muito reduzida. Quando um like é dado apenas incrementa o contador de likes da disciplina e quando um comentário é feito o novo comentário fica concatenado com os comentários anteriores com um newline entre eles. Qualquer usuário logado pode dar mais um like e comentar na disciplina.

- Temos um arquivo [JSON](./disciplinas.json) já com os nomes de todas as disciplinas que devem ser criadas. A ideia é programar sua API para povoar o banco de dados com todas as disciplinas já existentes neste arquivo. [Neste documento](http://bit.ly/inicia-dados-json) estão as dicas de como fazer isso usando spring boot. Lembrando que a própria API deve se encarregar de gerar os identificadores únicos das disciplinas no banco. Com isso, não precisaremos mais de uma rota na API para adicionar disciplinas. Outro lembrete: essa atividade envolve já o uso do banco, então você deve já ter criado o repositório de Disciplinas, e já deve ter configurado o banco em application.properties. (para testar você pode usar a rota GET /v1/api/disciplinas que retornará todas as disciplinas inseridas no sistema).

## Use spring boot e java para desenvolver a seguinte API:

- `POST /usuarios` (adiciona um usuario com email, nome e senha - o email é o login do usuario e deve ser um identificador único na tabela de usuário);

- `POST /auth/login` - recebe email e senha de um usuário, verifica na base de dados de usuários se esse usuário existe, e se a senha está correta (autenticação). Se o usuário for autenticado gerar um JWT que deve ser retornado para o cliente.

- `DELETE /auth/usuarios/` Remove o usuário cujo que está requisitando a deleção (esta identificação é feita através do token passado no authorization header da requisição HTTP). Retorna informação do usuário removido (em um JSON no corpo da resposta) e código 200. Esta ação só pode ser realizada pelo próprio usuário que quer se remover, assim é preciso receber um JWT na requisição e recuperar credenciais do usuário. Retornar código HTTP adequado para as possíveis possibilidades (ex. requisição sem JWT, com JWT inválido, ou com JWT válido mas de outro usuário).

- `GET /api/disciplinas` Retorna um JSON (com campos id, nome) com todas as disciplinas inseridas no sistema e código 200. Precisa estar logado para recuperar esta informação do sistema.

- `GET /api/disciplinas/{id}` Retorna um JSON que representa a disciplina completa (id, nome, nota, likes e comentarios) cujo identificador único é id e código 200. Ou não retorna JSON e código 404 (not found) caso o id passado não tenha sido encontrado. 

- `PUT /api/disciplinas/likes/{id}` Incrementa em um o número de likes da disciplina. O usuário deve estar logado para acessar este recurso.
Retorna a disciplina que foi atualizada (incluindo o id, nome e likes) e código 200. Ou não retorna JSON e código 404 (not found) caso o id passado não tenha sido encontrado. Se o usuário não estiver logado ou se o token estiver expirado código HTTP UNAUTHORIZED deve ser retornado.

- `PUT /api/disciplinas/nota/{id}` Atualiza a nota da disciplina de identificador id no sistema. No corpo da requisição HTTP deve estar um JSON com a nova nota da disciplina a ser atualizada no sistema. 
Retorna a disciplina que foi atualizada (incluindo o id, nome e nota) e código 200. Ou não retorna JSON e código 404 (not found) caso o id passado não tenha sido encontrado. Se o usuário não estiver logado ou se o token estiver expirado código HTTP UNAUTHORIZED deve ser retornado.

- `PUT /api/disciplinas/comentarios/{id}` Insere um novo comentário na disciplina de identificador id. No corpo da requisição HTTP deve estar um JSON com o novo comentário (chave "comentario") a ser adicionado na disciplina a ser atualizada no sistema. Retorna a disciplina que foi atualizada (incluindo o id, nome e os comentarios atualizados) e código 200. Ou não retorna JSON e código 404 (not found) caso o id passado não tenha sido encontrado. Se o usuário não estiver logado ou se o token estiver expirado código HTTP UNAUTHORIZED deve ser retornado.

- `GET /api/disciplinas/ranking/notas` Retorna todas as disciplinas inseridas no sistema ordenadas pela nota (da maior para a menor) e código 200.

- `GET /api/disciplinas/ranking/likes` Retorna todas as disciplinas inseridas no sistema ordenadas pelo número de likes (da que tem mais likes para a que tem menos likes) e código 200.

## Seguem algumas dicas:

* Use o **padrão DAO** para acesso às bases de dados;
* Desenvolva tudo sem a parte de autenticação/autorização, depois que estiver tudo testado adiciona a parte de **JWT**;
* Use **JWT** para autenticação e autorização;
* Siga **boas práticas de design**, buscando desacoplamento utilize corretamente **controladores, serviços e repositórios**;
* Organize suas **classes em packages** com nomes significativos (xx.services, xx.controllers, xx.repositories, xx.entities, etc. - pode usar nomes em portugues também, mas mantenha a coerência, ou tudo em portugues ou tudo em ingles);

## Executar a sua aplicação:

- Execute a sua aplicação no termina, dentro do diretório raiz do seu projeto com o seguinte comando: 
~~~shel
$ ./mvnw spring-boot:run
~~~

- Use Curl ou Postman para testar sua API. Para testar chamadas que requerem um JWT você deve copiar o token recebido na respostqa do login no header "Authorization" de uma requisição HTTP futura. 

**Não faça tudo de uma vez**. Desenvolva uma funcionalidade, teste, vá para a próxima…
