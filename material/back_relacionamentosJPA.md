# Entendendo um pouco mais de JPA e relacionamentos entre entidades

Já vimos que precisamos criar classes de modelo, conhecidas como entidades, para representar as classes do domínio da aplicação sendo desenvolvida. Se vamos vender bananas em navios, certamente teremos entidades para representar bananas e entidades para representar navios... Estas classes são marcadas com a anotação @Entity. Estas entidades precisam obrigatoriamente ter um atributo que serve como identificador único, que anotamos com @Id.

Entender os relacionamentos entre as entidades é essencial para modelar qualquer aplicação. Como desenvolvedor, na maioria das vezes, você encontrará um relacionamento ManyToOne / OneToMany entre entidades e o quão bem esses relacionamentos são mapeados/configurados determina em grande parte a complexidade da sua aplicação.

Em uma API certamente teremos várias classes de entidade. Em se tratando de JPA (Java Persistence API) contamos com um componente chamado EntityManager, que é parte da Java Persistence API. A função do EntityManager é implementar as interfaces  e as regras de ciclo de vida definidas pela especificação JPA 2.0. É esta entidade então que vai lidar com as várias entidades @Entity gerenciando seu ciclo de vida e garantindo sua persistência no banco de dados como tabelas.

Neste módulo vamos entender um pouco melhor sobre relacionamentos entre entidades. É possível relacionar duas entidades através de uma das seguintes anotações:

* @OneToOne: é um relacionamento um para um que define que só há uma entidade relacionada com a outra. Um exemplo desse tipo de relacionamento pode ocorrer entre a entidade Loja e a entidade endereço. Suponha que a aplicação comporta uma rede de lojas, cada uma com seu endereço. Ao configurar este relacionamento estamos dizendo que toda loja tem um endereço e todo endereço só pertence a uma loja.
* @OneToMany: só deve existir uma entidade da classe onde esta relação é definida, mas esta entidade está associada a muitas entidades da outra classe. Essa relação vai ocorrer sempre que uma classe tem uma coleção de tipos de outra classe. Esta é a associação entre uma cesta de compras e os produtos adicionados nela. Uma cesta de compras deve poder estar associada a muitos produtos, mas um produto específico do estoque só pode estar associado a uma cesta de compras (caso contrario estariamos vendendo o mesmo produto a usuários diferentes).
* @ManyToOne: é a relação inversa a @OneToMany, deve especificar a entidade que é a parte muitos. Por exemplo, na classe Produto, teriamos uma associação @ManyToOne com a classe CestaDeCompras.
* @ManyToMany: é uma associação de muitos para muitos. É usada nos casos em que uma entidade de um tipo A pode estar associada a muitas entidades do tipo B e cada entidade do tipo B também pode estar associada a várias entidades do tipo A. Imagine por exemplo, a relação entre livros e autores. Um livro pode ter muitos autores e um autor pode escrever muitos livros.

![Relações entre entidades](https://port.sas.ac.uk/pluginfile.php/233/mod_book/chapter/140/10%20Image%20D3.jpg)
> Fonte: https://port.sas.ac.uk/mod/book/view.php?id=75&chapterid=140

## Configurando relações

Na associação entre entidades vai sempre existir a **entidade proprietária** e a entidade não proprietária. A entidade proprietária da relação é aquela entidade que possui a chave estrangeira da outra entidade. Em geral, a entidade que está do lado \*ToOne será a proprietária. Vejamos um exemplo:

Vamos imaginar que estamos escrevendo a API para comércio eletrônico e precisamos modelar produtos (que representam produtos físicos em estoque que podem ser vendidos na loja) e a cesta de compras do usuário. Nos pedaços de código abaixo configuramos uma relação entre CestaDeCompras e Produto.

````java
@Entity
public class Produto {
  @Id
  private Long idProduto;
  
  ...
  
  @ManyToOne(fetch = FetchType.LAZY, optional = false)
  @JoinColumn(name = "idCesta")
  @JsonIgnore
  private CestaDeCompras cesta;
  ...
}
````
Esse primeiro trecho de código nos informa através da anotação @ManyToOne que aparece antes do atributo cesta que estamos configurando um relacionamento muitos para um que reflete a seguinte regra: muitos produtos podem estar associados à mesma cesta de compras. Faz sentido? Sim, é a regra de negócio comum: um usuário pode colocar quantos produtos desejar em sua cesta de compras. Só com esta associação já temos uma relação definida, mesmo que nada seja configurado na cesta de compras. Com esta anotação, o que acontece "por trás" no banco de dados é que na tabela PRODUTO vai existir uma chave estrangeira que será o ID da cesta de compras associada ao produto. A entidade Produto será a entidade proprietária da relação, uma vez que é em sua tabela que está a chave estrangeira. 

Dizemos que é uma relação **unidirecional**, pois só a entidade Produto sabe da existência dessa relação. Através desta relação conseguimos recuperar, apenas a partir da entidade produto, a cesta de compras relacionada. Isto significa que ao recuperar o ID de uma cesta de compras precisaremos de uma query extra para recuperar os produtos da cesta a partir da tabela de produtos. Para recuperar os produtos da cesta de compras teremos que pesquisar todos os produtos associados ao ID da cesta de interesse.

Para que isso seja realizado de forma simples é preciso acrescentar no repositório de Produto um método que retorne todos os produtos associados a um dado ID de cesta de compras. Mais adiante, quando entendermos melhor toda a configuração específica desta relação, mostramos como ficaria o DAO.

Temos a chance de tornar esta relação **bidirecional** informando na entidade CestaDeCompras sobre a relação. Nesse caso será possível acessar, a partir da cesta de compras, os produtos relacionados a ela. A configuração da classe CestaDeCompras está apresentada no trecho de código abaixo.

````java
@Entity
public class CestaDeCompras {
  @Id
  private Long idCesta;
  
  ...
  
  @OneToMany(mappedBy = "cesta", fetch = FetchType.EAGER)
  private List<Produto> produtos;
  ...
}
````

Nesse trecho de código usamos a anotação @OneToMany no atributo List<Produto> produtos. Esta anotação define uma regra: uma cesta de compras pode estar relacioanda a muitos produtos. Ao configurar a relação também na classe CestaDeCompras tornamos a relação bidirecional e é possivel recuperar os produtos a partir também da cesta de compras.
  
Ao estabelecer esta relação bidirecional, o que o JPA realiza no contexto do banco de dados é muito parecido com o que já tínhamos no contexto da relação unidirecional. São duas tabelas: CESTA_DE_COMPRAS e PRODUTO. A tabela CESTA_DE_COMPRAS tem as colunas ID_CESTA, e outras colunas relativas à cesta. Na tabela PRODUTO temos as colunas ID_PRODUTO, outras colunas de produto e a coluna ID_CESTA que é a chave estrangeira da tabela CESTA_DE_COMPRAS. Esta coluna é a que permite o *join* entre a CESTA_DE_COMPRAS e PRODUTO. Como a chave estrangeira está na entidade Produto, dizemos que Produto é a entidade proprietária dessa relação. Até aqui nenhuma novidade em relação ao que já tínhamos. O que muda é o seguinte: com a relação bidirecional, ao recuperar uma cesta de compras, a coleção de produtos associada a ela já vem no objeto da classe CestaDeCompras. Ao contrário, quando a relação era unidirecional, a lista de produtos não vinha na cesta. Era preciso identificar o ID da cesta e realizar nova query para recuperar todos os produtos associados ao ID_CESTA de interesse.

Em resumo: relacionamentos podem ser bidirecionais ou unidirecionais. Em uma relação unidirecional apenas um lado da relação conhece a relação. O outro lado não sabe que a relação existe. Sendo assim, usamos a anotação de relacionamento (@OneToOne, @OneToMany, etc.) em apenas uma entidade da relação, a entidade que fica ciente da relação. Ao contrário, na relação bidirecional ambas as entidades sabem da existência da relação. Nas relações bidirecionais podemos navegar nas entidades nas duas direções sem precisar de queries extra.

## Configuração de Cascata (Cascade)

JPA permite operações em cascata que se propagam da entidade "mãe" para a "filha". Tipicamente, a entidade mãe é a não proprietária. No nosso exemplo, a entidade mãe é a CestaDeCompras, que tem a associação com a entidade filha Produto. As operações SELECT, INSERT, UPDATE, DELETE podem ser propagadas para a base de dados da entidade filha.

Os tipos de cascata JPA são PERSIST, MERGE, REFRESH, REMOVE, DETACH, ALL. Ao realizar estas operações na entidade mãe, a entidade filha também deve ser modificada para refletir as mudanças. Essa configuração é muito específica do tipo de relacionamento e não existe uma configuração default única. Para relacionamentos bidirecionais OneToOne, por exemplo, faz sentido o CascadeType.ALL uma vez que o ciclo de vida de uma entidade está associada ao ciclo de vida da outra. 

Em uma relação bidirecional como a que exemplificamos entre CestaDeCompras e Produto, se configurarmos na CestaDeCompras o CascadeType.ALL indicamos que todos os produtos associados à cesta sejam removidos da tabela de produtos, o que talvez não seja o que desejamos... No entando em uma relação semelhante entre Post e ComentariosDoPost provavelmente queremos que todos os comentários sejam removidos ao remover um post.

A dica é entender a relação e definir a melhor configuração para cada caso. Na dúvida faça testes simples que lhe permitam entender sua configuração e como as operações em cascata estão funcionando.

Uma configuração que anda perto da CascadeType.REMOVE é a orphanRemove = true. Esta opção marca a entidade filha a ser removida se ela não tiver mais referências a partir da entidade mãe. Por exemplo, no relacionamento entre Comentarios de um Post e o Post, inserir esta configuração indica que se o Post deixar de existir ou se fizermos um post.setComentarios(null) todos os comentários do post devem ser removidos, pois não existe mais nenhuma referência a estes e eles podem deixar de existir. No nosso exemplo de produto e cesta de compras, será que queremos que os produtos sejam removidos quando fizermos cesta.setProdutos(null)?

## Modo de busca (FETCH)

Uma outra configuração possível diz respeito à forma como os dados são recuperados do banco de dados. Essa é a configuração *FETCH*. Existem duas estratégias para esta configuração:

* A estratégia *EAGER* indica que em tempo de execução os dados devem ser recuperados em uma consulta de uma vez. Isto significa que se a estratégia EAGER for usada, o EntityManager vai recuperar os dados da entidade mãe e da entidade filha de uma só vez por uma consulta. 
* A estratégia *LAZY* indica que os dados serão obtidos de forma preguiçosa. isto significa que eles devem ser recuperados apenas quando forem acessados pela primeira vez. Neste caso, os dados são recuperados quando necessário através de subconsultas. O EntityManager recupera os dados da entidade mãe primeiro e depois os dados da entidade filha sob demanda. No caso do exemplo da relação entre cesta de compras e produto. Ao usar a estratégia EAGER ao recuperar uma cesta de compras, os produtos associados já são também recuperados. Já se a estratégia LAZY for usada, ao recuperar um produto apenas os atributos de produto específicos são recuperados. Apenas com uma chamada a, por exemplo, cesta.getProdutos() que os produtos associados à cesta de compras seriam recuperados do banco de dados.

## Sobre os repositórios

Alguns métodos adicionais podem ser úteis quando falamos de relacionamentos OneToMany. Por exemplo, recuperar a partir da tabela de produtos todos os produtos de uma dada cesta de compras (temos que conhecer o ID da cesta). Em se tratando da interface JPARepository, criamos novos métodos seguindo regras estritas para nomeação/assinatura do método novo. A consulta ao banco de dados será criada automaticamente ao seguir as regras para nomeação do método. 

O código abaixo para a interface ProdutosDAO adiciona à interface um novo metodo que retorna todos os produtos que estão associados à cesta de compras com o ID passado como parâmetro.

````java
@Repository
public interface ProdutosDAO<T, ID extends Serializable> extends JpaRepository<Produto, Long> {

	List<Comentario> findByCestaIdCesta(Long id);
}
````
Este método irá recuperar todos os registros na tabela de produtos cujo atributo idCesta (que vem da associação cesta) seja igual ao ID passado. Para entender melhor: esse nome só funciona porque na classe Produto existe o atributo cesta definido como relação de muitos para um entre a classe Produto e a classe CestaDeCompras:

````java
  @ManyToOne(fetch = FetchType.LAZY, optional = false)
  @JoinColumn(name = "idCesta")
  @JsonIgnore
  private CestaDeCompras cesta;
````

Nessa mesma configuração damos um nome à coluna que servirá de *join* para esta associação, o nome que demos foi idCesta. Então estamos dizendo que recuperamos a cesta de compras associada ao produto através do id da cesta. Em termos de banco de dados, o que acontece é que na tabela de PRODUTO vai haver uma coluna chamada ID_CESTA que é a chave estrangeira de CestaDeCompras na tabela PRODUTO. São essas configurações que usamos para gerar o nome do método e derivar a consulta ao banco automaticamente (sem precisar escrever uma @Query explícita). Se o atributo que chamamos cesta fosse chamado cestaDeCompras, então o nome do método na interface mudaria para findByCestaDeComprasIdCesta.

Mais detalhes de como gerar os nomes dos métodos para derivar as consutas podem ser vistos [aqui](https://www.baeldung.com/spring-data-derived-queries), [aqui](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.query-methods.query-creation) e [aqui](https://www.baeldung.com/spring-data-sorting).

## Documentação de referência

[Repositórios JPA](https://docs.spring.io/spring-data/jpa/docs/1.5.0.RELEASE/reference/html/index.html)

[Tutorial java sobre persistência no backend](https://docs.oracle.com/javaee/5/tutorial/doc/bnbrs.html)

[javadoc da JPA](https://docs.oracle.com/javaee/7/api/javax/persistence/package-summary.html)

[JPA mini book](http://enos.itcollege.ee/~jpoial/java/naited/JPA_Mini_Book.pdf)

[Artigos avançados relacionados a JPA da baeldung](https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)

[Dicas para criação de queries a partir dos nomes dos métodos](https://www.baeldung.com/spring-data-derived-queries)

