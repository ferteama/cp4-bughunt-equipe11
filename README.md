# Checkpoint 4 — Bug Hunt StreamFIAP

## Identificação

**Grupo:** Equipe11

| Integrante | RM | Turma |
|---|---|---|
| Auro Vanetti | RM563761 | 2CCPH |
| Renan Mano Otero | RM554911 | 2CCPH |
| Marco Antonio Ferreira Fonseca | RM566434 | 2CCPH |
| Bruno Soares de Santanna | RM562235 | 2CCPH |
| Enzo Yokokura Araujo | RM564177 | 2CCPH |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | ___ / 12 |
| **Total de ajustes de Clean Code** | ___ / 6 |

---

## Parte 1 — Bugs encontrados

> Uma linha por bug, na ordem em que você os encontrou. Use a numeração dos seus
> commits (`fix: bug01 ...`). Preencha TODAS as colunas — metade da nota está aqui.

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | Falta da indicação da variável Nome em Usuario.class | Linha 22 do Usuario.class | Adicionar "this." a variável | Revisão - Exercício ENADE: Questão 3 |
| bug02 | Sistema acha que o usuário tem dinheiro que não tem | Linha 28 do Usuario.class | Trocar lugar de this.creditos e preco | Comparação de valores |
| bug03 | Todos usuários possuem ID null | Linha 12 do Usuario.class | Adicionamos @GeneratedValue(strategy = GenerationType.IDENTITY) | Atributo não definido |
| bug04 | Comparação incorreta de String utilizando "==" | Linha 47 do ConteudoController.java | Substituímos para .equals() para comparar as strings | Comparação entre strings |
| bug05 | O método captura qualquer exceção e retorna null | Linha 30 do ConteudoController.java | Removemos o try/catch e simplificamos o retorno para ResponseEntity.ok(conteudo) | Tratamento de exceções |
| bug06 | O sistema permitia alugar um conteúdo mesmo quando ele estava indisponível. | Método alugar() do AluguelController.java, após a busca do conteúdo | Adicionamos uma verificação com if (!conteudo.isDisponivel()) e lançamos a ConteudoIndisponivelException quando o conteúdo não estiver disponível para aluguel | Validação de regras de negócio e tratamento de exceções |
| bug07 | O valor do aluguel aumentava em 20% ao aplicar a promoção, em vez de receber 20% de desconto | Linha 25 do Filme.java | Alteramos return preco * 1.2 para return preco * 0.8, fazendo com que o preço seja reduzido em 20% | Cálculo de porcentagem |
| bug08 | Ao cadastrar uma série, ela ficava sem título, categoria e duração (tudo salvo como null/0) | Construtor Serie(...) em Serie.java, linhas 14–16 | Adicionamos a chamada super(titulo, categoria, duracaoMinutos, classificacaoEtaria, true) para inicializar os atributos herdados de Conteudo | Herança / Construtores |
| bug09 | Séries eram cobradas sempre 9.90 (preço padrão), ignorando o número de temporadas | Linha 20 do Serie.java | Removemos o parâmetro desconto e adicionamos @Override, transformando a sobrecarga em sobrescrita real do método de Conteudo | Sobrescrita vs sobrecarga de métodos |
| bug10 |Ao tentar alugar um conteúdo com classificação etária incompatível, a API retornava erro 500 genérico em vez de uma mensagem clara. |GlobalExceptionHandler.java — faltava tratamento para ClassificacaoIndicativaException |Criamos o método handleClassificacaoIndicativa com @ExceptionHandler, retornando 403 com a mensagem da regra |Exceções checked vs unchecked / Tratamento de exceções |
| bug11 | Era possível cadastrar um usuário com créditos negativos via POST /api/usuarios. | Construtor Usuario(...) em Usuario.java, linhas 22–26 | Adicionamos validação if (creditos < 0) throw new IllegalArgumentException(...) antes de atribuir o valor | Validação de regras de negócio / Encapsulamento |
| bug12 |Buscar um usuário inexistente (GET/api/usuarios/{id}) retornava página de erro HTML padrão (500) em vez de um JSON com status 404. |UsuarioController.java/AluguelController.java usam IllegalArgumentException, sem handler em GlobalExceptionHandler.java | Adicionamos handleIllegalArgument mapeando IllegalArgumentException para status 404 com mensagem em JSON | Tratamento de exceções |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 |Método alugar() em Usuario.java | Nomes significativos - A variável p não dizia nada sobre seu conteúdo |Renomeei a variável p para preco em todas as suas ocorrências no método |
| clean02 | Método debitarCreditos() em Usuario.java |Comentários devem dizer a verdade sobre o código — o comentário dizia "adiciona", mas o código subtrai | Corrigi o texto do comentário para refletir o que o código realmente faz |
| clean03 | Final da classe ConteudoController.java | Evitar código morto/comentado (Clean Code) — método não utilizado e bloco de comentário com código antigo | Removi o método calcularDescontoAntigo() e o bloco de código comentado (histórico fica no Git, não no fonte) |
| clean04 | Atributo duracaoMinutos em Conteudo.java e seus usos em ConteudoController.java | Encapsulamento — atributo público permitia acesso direto, ignorando o getter/setter | Tornei o campo private e substituí os acessos diretos (filme.duracaoMinutos) pelo método getDuracaoMinutos() |
| clean05 | Método alugar() em Usuario.java | Responsabilidade única / SRP  — o método misturava regra de negócio do aluguel com a impressão do recibo | Extraí a impressão do recibo para o método privado imprimirRecibo(), deixando alugar() focado só na regra de negócio |
| clean06 | | | |

---

## Parte 3 — Perguntas de reflexão

> Responda com suas palavras, 5 a 10 linhas cada, **usando o código real do projeto
> como exemplo**. Respostas genéricas de tutorial não pontuam.

### 1. Injeção de dependência (Aula 13)
Os controllers recebem os repositories via `@Autowired` (ex.: `ConteudoController`
usa `ConteudoRepository`). Explique por que o Spring precisa gerenciar esses objetos
em vez de criarmos com `new ConteudoRepository()`. O que exatamente o Spring faz ao
injetar um bean, e por que isso não funcionaria com um `new` comum?

### 2. JDBC vs Spring Data JPA (Aulas 12 e 13)
Na Aula 12 escrevemos um `ProdutoDAO` na mão com `Connection`, `PreparedStatement` e
`ResultSet`. Aqui o `ConteudoRepository` tem 2 linhas e faz CRUD completo. Compare as
duas abordagens: o que o Spring Data JPA automatiza, o que o JDBC/DAO ainda resolve
melhor, e como o `findByCategoria` consegue funcionar sem implementação.

### 3. Exceções checked vs unchecked (Aula 11)
A `ClassificacaoIndicativaException` estourava como um erro genérico do servidor,
sem mensagem útil para o cliente. Explique a diferença entre `extends Exception` e
`extends RuntimeException` no contexto desse bug, e como você fez a mensagem da
regra (classificação indicativa) chegar de forma clara ao cliente da API.

### 4. Sobrescrita vs sobrecarga (Aula 7)
Um dos bugs compilava sem nenhum erro: o método da `Serie` parecia sobrescrever
`calcularPrecoAluguel`, mas na verdade sobrecarregava. Explique a diferença entre
override e overload nesse caso e por que a anotação `@Override` teria impedido o bug.

### 5. Onde blindar o objeto? (Aulas 3, 4 e 13)
Vimos bugs de dados inválidos aceitos (duração negativa, créditos negativos, campos
nulos). Em quais lugares (construtor, setter, método do model) cada tipo de validação
deve ficar? Justifique usando os bugs que você encontrou e explique por que validar só
em um lugar não foi suficiente.

### 6. Abstração e interface (Aulas 8 e 9)
`Conteudo` é abstrata e `Promocionavel` é uma interface. Explique a diferença de
propósito entre as duas nesse projeto e o que mudaria no código se o Documentário
passasse a ter promoções — quais classes/linhas seriam tocadas e quais ficariam
intactas? O que isso diz sobre o design do sistema?

---

## Parte 4 — Espaço livre (opcional)

Alguma dificuldade, dúvida ou comentário sobre o checkpoint?

```

```
