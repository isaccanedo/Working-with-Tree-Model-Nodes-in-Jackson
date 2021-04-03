## Jackson Cookbooks and Examples

# 1. Visão Geral
Este tutorial se concentrará em trabalhar com nós de modelo de árvore em Jackson.

Usaremos JsonNode para várias conversões, bem como adicionar, modificar e remover nós.

# 2. Criação de um nó
A primeira etapa na criação de um nó é instanciar um objeto ObjectMapper usando o construtor padrão:

```
ObjectMapper mapper = new ObjectMapper();
```

Como a criação de um objeto ObjectMapper é cara, é recomendado que o mesmo seja reutilizado para várias operações.

Em seguida, temos três maneiras diferentes de criar um nó de árvore, uma vez que temos nosso ObjectMapper.

### 2.1. Construir um Nó do zero
A maneira mais comum de criar um nó do nada é a seguinte:

```
JsonNode node = mapper.createObjectNode();
```

Como alternativa, também podemos criar um nó por meio do JsonNodeFactory:

```
JsonNode node = JsonNodeFactory.instance.objectNode();
```

### 2.2. Analisar de uma fonte JSON
Esse método é bem abordado no artigo Jackson - Marshall String to JsonNode. Consulte-o se precisar de mais informações.

### 2.3. Converter de um objeto
Um nó pode ser convertido de um objeto Java chamando o método valueToTree (Object fromValue) no ObjectMapper:

```
JsonNode node = mapper.valueToTree(fromValue);
```

A API convertValue também é útil aqui:

```
JsonNode node = mapper.convertValue(fromValue, JsonNode.class);
```

Vamos ver como funciona na prática. Suponha que temos uma classe chamada NodeBean:

```
public class NodeBean {
    private int id;
    private String name;

    public NodeBean() {
    }

    public NodeBean(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // standard getters and setters
}
```

Vamos escrever um teste para garantir que a conversão aconteça corretamente:

```
@Test
public void givenAnObject_whenConvertingIntoNode_thenCorrect() {
    NodeBean fromValue = new NodeBean(2016, "baeldung.com");

    JsonNode node = mapper.valueToTree(fromValue);

    assertEquals(2016, node.get("id").intValue());
    assertEquals("baeldung.com", node.get("name").textValue());
}
```

# 3. Transformando um Nó
### 3.1. Escreva como JSON
O método básico para transformar um nó de árvore em uma string JSON é o seguinte:

```
mapper.writeValue(destination, node);
```

onde o destino pode ser um arquivo, um OutputStream ou um gravador.
Ao reutilizar a classe NodeBean declarada na seção 2.3, um teste garante que esse método funcione conforme o esperado:

```
final String pathToTestFile = "node_to_json_test.json";

@Test
public void givenANode_whenModifyingIt_thenCorrect() throws IOException {
    String newString = "{\"nick\": \"cowtowncoder\"}";
    JsonNode newNode = mapper.readTree(newString);

    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ((ObjectNode) rootNode).set("name", newNode);

    assertFalse(rootNode.path("name").path("nick").isMissingNode());
    assertEquals("cowtowncoder", rootNode.path("name").path("nick").textValue());
}
```

### 3.2. Converter em objeto
A maneira mais conveniente de converter um JsonNode em um objeto Java é a API treeToValue:

```
NodeBean toValue = mapper.treeToValue(node, NodeBean.class);
```

O que é funcionalmente equivalente a:

```
NodeBean toValue = mapper.convertValue(node, NodeBean.class)
```

Também podemos fazer isso por meio de um fluxo de token:

```
JsonParser parser = mapper.treeAsTokens(node);
NodeBean toValue = mapper.readValue(parser, NodeBean.class);
```

Por fim, vamos implementar um teste que verifica o processo de conversão:

```
@Test
public void givenANode_whenConvertingIntoAnObject_thenCorrect()
  throws JsonProcessingException {
    JsonNode node = mapper.createObjectNode();
    ((ObjectNode) node).put("id", 2016);
    ((ObjectNode) node).put("name", "baeldung.com");

    NodeBean toValue = mapper.treeToValue(node, NodeBean.class);

    assertEquals(2016, toValue.getId());
    assertEquals("baeldung.com", toValue.getName());
}
```

# 4. Manipulando nós de árvore

Os seguintes elementos JSON, contidos em um arquivo denominado example.json, são usados como uma estrutura de base para as ações discutidas nesta seção a serem realizadas:

```
{
    "name": 
        {
            "first": "Tatu",
            "last": "Saloranta"
        },

    "title": "Jackson founder",
    "company": "FasterXML"
}
```

Este arquivo JSON, localizado no caminho de classe, é analisado em uma árvore modelo:

```
public class ExampleStructure {
    private static ObjectMapper mapper = new ObjectMapper();

    static JsonNode getExampleRoot() throws IOException {
        InputStream exampleInput = 
          ExampleStructure.class.getClassLoader()
          .getResourceAsStream("example.json");
        
        JsonNode rootNode = mapper.readTree(exampleInput);
        return rootNode;
    }
}
```

Observe que a raiz da árvore será usada ao ilustrar as operações nos nós nas subseções a seguir.

### 4.1. Localizando um Nó
Antes de trabalhar em qualquer nó, a primeira coisa que precisamos fazer é localizá-lo e atribuí-lo a uma variável.

Se o caminho para o nó for conhecido de antemão, isso é muito fácil de fazer. Por exemplo, digamos que queremos um nó chamado último, que está sob o nome de nó:

```
JsonNode locatedNode = rootNode.path("name").path("last");
```

Como alternativa, get ou with APIs também podem ser usados em vez de path.

Se o caminho não for conhecido, a pesquisa, é claro, se tornará mais complexa e iterativa.

Podemos ver um exemplo de iteração sobre todos os nós em 5. Iteração sobre os nós

### 4.2. Adicionando um Novo Nó
Um nó pode ser adicionado como filho de outro nó da seguinte maneira:

```
ObjectNode newNode = ((ObjectNode) locatedNode).put(fieldName, value);
```

Muitas variantes sobrecarregadas de put podem ser usadas para adicionar novos nós de diferentes tipos de valor.

Muitos outros métodos semelhantes também estão disponíveis, incluindo putArray, putObject, PutPOJO, putRawValue e putNull.

Finalmente - vamos dar uma olhada em um exemplo - onde adicionamos uma estrutura inteira ao nó raiz da árvore:

```
"address":
{
    "city": "Seattle",
    "state": "Washington",
    "country": "United States"
}
```

Aqui está o teste completo passando por todas essas operações e verificando os resultados:

```
@Test
public void givenANode_whenAddingIntoATree_thenCorrect() throws IOException {
    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ObjectNode addedNode = ((ObjectNode) rootNode).putObject("address");
    addedNode
      .put("city", "Seattle")
      .put("state", "Washington")
      .put("country", "United States");

    assertFalse(rootNode.path("address").isMissingNode());
    
    assertEquals("Seattle", rootNode.path("address").path("city").textValue());
    assertEquals("Washington", rootNode.path("address").path("state").textValue());
    assertEquals(
      "United States", rootNode.path("address").path("country").textValue();
}
```

### 4.3. Editando um Nó
Uma instância ObjectNode pode ser modificada invocando o método set (String fieldName, JsonNode value):

```
JsonNode locatedNode = locatedNode.set(fieldName, value);
```

Resultados semelhantes podem ser obtidos usando os métodos replace ou setAll em objetos do mesmo tipo.

Para verificar se o método funciona conforme o esperado, alteraremos o valor do nome do campo sob o nó raiz de um objeto de primeiro e último para outro que consiste em apenas um campo de nick em um teste:

```
@Test
public void givenANode_whenModifyingIt_thenCorrect() throws IOException {
    String newString = "{\"nick\": \"cowtowncoder\"}";
    JsonNode newNode = mapper.readTree(newString);

    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ((ObjectNode) rootNode).set("name", newNode);

    assertFalse(rootNode.path("name").path("nick").isMissingNode());
    assertEquals("cowtowncoder", rootNode.path("name").path("nick").textValue());
}
```

### 4.4. Removendo um Nó
Um nó pode ser removido chamando a API remove (String fieldName) em seu nó pai:

```
JsonNode removedNode = locatedNode.remove(fieldName);
```

Para remover vários nós de uma vez, podemos invocar um método sobrecarregado com o parâmetro do tipo Collection <String>, que retorna o nó pai em vez daquele a ser removido:

```
ObjectNode locatedNode = locatedNode.remove(fieldNames);
```

No caso extremo, quando queremos excluir todos os subnós de um determinado nó - a API removeAll é útil.

O teste a seguir se concentrará no primeiro método mencionado acima - que é o cenário mais comum:

```
@Test
public void givenANode_whenRemovingFromATree_thenCorrect() throws IOException {
    JsonNode rootNode = ExampleStructure.getExampleRoot();
    ((ObjectNode) rootNode).remove("company");

    assertTrue(rootNode.path("company").isMissingNode());
}
```

# 5. Iterando sobre os nós
Vamos iterar todos os nós em um documento JSON e reformatá-los em YAML. JSON tem três tipos de nó, que são Value, Object e Array.

Portanto, vamos garantir que nossos dados de amostra tenham todos os três tipos diferentes, adicionando um Array:

```
{
    "name": 
        {
            "first": "Tatu",
            "last": "Saloranta"
        },

    "title": "Jackson founder",
    "company": "FasterXML",
    "pets" : [
        {
            "type": "dog",
            "number": 1
        },
        {
            "type": "fish",
            "number": 50
        }
    ]
}
```

Agora, vamos ver o YAML que queremos produzir:

```
name: 
  first: Tatu
  last: Saloranta
title: Jackson founder
company: FasterXML
pets: 
- type: dog
  number: 1
- type: fish
  number: 50
```

Sabemos que os nós JSON possuem uma estrutura de árvore hierárquica. Portanto, a maneira mais fácil de iterar em todo o documento JSON é começar na parte superior e ir descendo por todos os nós filhos.

Vamos passar o nó raiz para um método recursivo. O método irá então chamar a si mesmo com cada filho do nó fornecido.

### 5.1. Testando a Iteração
Começaremos criando um teste simples que verifica se podemos converter JSON em YAML com sucesso.

Nosso teste fornece o nó raiz do documento JSON para nosso método toYaml e afirma que o valor retornado é o que esperamos:

```
@Test
public void givenANodeTree_whenIteratingSubNodes_thenWeFindExpected() throws IOException {
    JsonNode rootNode = ExampleStructure.getExampleRoot();
    
    String yaml = onTest.toYaml(rootNode);

    assertEquals(expectedYaml, yaml); 
}

public String toYaml(JsonNode root) {
    StringBuilder yaml = new StringBuilder(); 
    processNode(root, yaml, 0); 
    return yaml.toString(); }
}
```

### 5.2. Lidando com diferentes tipos de nós

Precisamos lidar com diferentes tipos de nó de maneira ligeiramente diferente. Faremos isso em nosso método processNode:

```
private void processNode(JsonNode jsonNode, StringBuilder yaml, int depth) {
    if (jsonNode.isValueNode()) {
        yaml.append(jsonNode.asText());
    }
    else if (jsonNode.isArray()) {
        for (JsonNode arrayItem : jsonNode) {
            appendNodeToYaml(arrayItem, yaml, depth, true);
        }
    }
    else if (jsonNode.isObject()) {
        appendNodeToYaml(jsonNode, yaml, depth, false);
    }
}
```

Primeiro, vamos considerar um nó Value. Simplesmente chamamos o método asText do nó para obter uma representação String do valor.

A seguir, vamos examinar um nó Array. Cada item dentro do nó Array é em si um JsonNode, portanto, iteramos sobre o Array e passamos cada nó para o método appendNodeToYaml. Também precisamos saber se esses nós fazem parte de um array.

Infelizmente, o nó em si não contém nada que nos diga isso, portanto, passaremos um sinalizador para nosso método appendNodeToYaml.

Finalmente, queremos iterar sobre todos os nós filhos de cada nó de objeto. Uma opção é usar JsonNode.elements. No entanto, não podemos determinar o nome do campo de um elemento, pois ele contém apenas o valor do campo:

```
Object  {"first": "Tatu", "last": "Saloranta"}
Value  "Jackson Founder"
Value  "FasterXML"
Array  [{"type": "dog", "number": 1},{"type": "fish", "number": 50}]
```

Em vez disso, usaremos JsonNode.fields, pois isso nos dá acesso ao nome e ao valor do campo:

```
Key="name", Value=Object  {"first": "Tatu", "last": "Saloranta"}
Key="title", Value=Value  "Jackson Founder"
Key="company", Value=Value  "FasterXML"
Key="pets", Value=Array  [{"type": "dog", "number": 1},{"type": "fish", "number": 50}]
```

Para cada campo, adicionamos o nome do campo à saída. Em seguida, processe o valor como um nó filho, passando-o para o método processNode:

```
private void appendNodeToYaml(
  JsonNode node, StringBuilder yaml, int depth, boolean isArrayItem) {
    Iterator<Entry<String, JsonNode>> fields = node.fields();
    boolean isFirst = true;
    while (fields.hasNext()) {
        Entry<String, JsonNode> jsonField = fields.next();
        addFieldNameToYaml(yaml, jsonField.getKey(), depth, isArrayItem && isFirst);
        processNode(jsonField.getValue(), yaml, depth+1);
        isFirst = false;
    }
        
}
```

Não podemos dizer a partir do nó quantos ancestrais ele tem. Portanto, passamos um campo chamado profundidade no método processNode para controlar isso. Incrementamos esse valor cada vez que obtemos um nó filho para que possamos recuar corretamente os campos em nossa saída YAML:

```
private void addFieldNameToYaml(
  StringBuilder yaml, String fieldName, int depth, boolean isFirstInArray) {
    if (yaml.length()>0) {
        yaml.append("\n");
        int requiredDepth = (isFirstInArray) ? depth-1 : depth;
        for(int i = 0; i < requiredDepth; i++) {
            yaml.append("  ");
        }
        if (isFirstInArray) {
            yaml.append("- ");
        }
    }
    yaml.append(fieldName);
    yaml.append(": ");
}
```

Agora que temos todo o código instalado para iterar nos nós e criar a saída YAML, podemos executar nosso teste para mostrar que funciona.

# 6. Conclusão

Este tutorial cobriu as APIs comuns e cenários de trabalho com um modelo de árvore em Jackson.