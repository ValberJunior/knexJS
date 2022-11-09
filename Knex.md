# Estudo Sobre o KnexJS <img src="https://knexjs.org/knex-logo.png" alt="img" style="zoom:20%;" />

<a href="https://knexjs.org/">Documentação completa</a>

:arrow_right: Instalação;

```sh
$ npm install knex --save

# Em seguida (adicionar a flag --save):
$ npm install pg
$ npm install pg-native
$ npm install sqlite3
$ npm install better-sqlite3
$ npm install mysql
$ npm install mysql2
$ npm install oracledb
$ npm install tedious
```

### Opções de configuração

O próprio **módulo** **Knex** é uma função que retorna um objeto de configuração para o **Knex**, aceitando alguns parâmetros. 

O parâmetro **client** é necessário e determina qual adaptador será usado com a biblioteca:

```javascript
const knex = require('knex')({
  client: 'mysql',
  connection: {
    host : '127.0.0.1',
    port : 3306,
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  }
});
```

As opções de conexão são passadas diretamente para o **client** **de banco de dados** apropriado para criar a conexão, e podem ser um objeto, uma cadeia de conexão ou uma função que devolva um objeto.



### PostgreSQL

O **client** **PostgreSQL** do Knex permite que que você defina o caminho de busca inicial para cada conexão automaticamente usando uma opção adicional "**searchPath**", como mostrado abaixo.

```js
const pg = require('knex')({
  client: 'pg',
  connection: process.env.PG_CONNECTION_STRING,
  searchPath: ['knex', 'public'],
});
```



### SQLite3 ou Better-SQLite3

Quando você usa o adaptador **SQLite3** ou **Better-SQLite3**, é necessário um nome de arquivo, não uma conexão de rede. Por exemplo:

```js
const knex = require('knex')({
  client: 'sqlite3', // or 'better-sqlite3'
  connection: {
    filename: "./mydb.sqlite"
  }
});
```

Você também pode executar **SQLite3** ou **Better-SQLite3** com um banco de dados na memória fornecendo `:memory:`como nome de arquivo. Por exemplo:

```js
const knex = require('knex')({
  client: 'sqlite3', // ou 'better-sqlite3'
  connection: {
    filename: ":memory:"
  }
});
```



### SQLite3

Ao usar o adaptador **SQLite3**, você pode definir os **sinalizadores** usados para abrir a conexão. Por exemplo:

```js
const knex = require('knex')({
  client: 'sqlite3',
  connection: {
    filename: "file:memDb1?mode=memory&cache=shared",
    flags: ['OPEN_URI', 'OPEN_SHAREDCACHE']
  }
});
```



## INFO

A versão do banco de dados pode ser adicionada na configuração do ***Knex***, quando você usa o adaptador **PostgresSQL** para conectar um banco de dados não padrão.



```js
const knex = require('knex')({
  client: 'pg',
  version: '7.2',
  connection: {
    host : '127.0.0.1',
    port : 3306,
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  }
});
```

```js
const knex = require('knex')({
  client: 'mysql',
  version: '5.7',
  connection: {
    host : '127.0.0.1',
    port : 3306,
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  }
});
```

Uma função pode ser usada para determinar a configuração da conexão dinamicamente. Essa função não recebe parâmetros e retorna um objeto de configuração ou uma **promise** para um objeto de configuração.

```js
const knex = require('knex')({
  client: 'sqlite3',
  connection: () => ({
    filename: process.env.SQLITE_FILENAME
  })
});
```

Por padrão, o objeto de configuração recebido por meio de uma função é armazenado em **cache** e reutilizado para todas as conexões. Para alterar esse comportamento, uma função **expirationChecker**  pode ser retornada como parte do objeto de configuração. O **expirationChecker** é consultado antes de tentar criar novas conexões , e caso retorne **true**, um novo objeto de configuração é recuperado. Por exemplo, para trabalhar com um token de autenticação com vida útil limitada:

```js
const knex = require('knex')({
  client: 'postgres',
  connection: async () => {
    const { 
      token, 
      tokenExpiration 
    } = await someCallToGetTheToken();

    return {
      host : 'your_host',
      port : 3306,
      user : 'your_database_user',
      password : token,
      database : 'myapp_test',
      expirationChecker: () => {
        return tokenExpiration <= Date.now();
      }
    };
  }
});
```

Você também pode se conectar por meio de um soquete de domínio **unix**, que ignorará o **host** e a porta.

```js
const knex = require('knex')({
  client: 'mysql',
  connection: {
    socketPath : '/path/to/socket.sock',
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  }
});
```

**Useparams** :arrow_right: É um parâmetro opcional que permite passar parâmetros arbitrários que serão acessíveis via propriedade **knex.userParams**.

```js
const knex = require('knex')({
  client: 'mysql',
  connection: {
    host : '127.0.0.1',
    port : 3306,
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  },
  userParams: {
    userParam1: '451'
  }
});

```

>**Inicializar a biblioteca normalmente deve acontecer apenas uma vez em seu aplicativo, pois cria um pool de conexões para o banco de dados atual, você deve usar a instância retornada da chamada de inicialização em toda a sua biblioteca.**

Especifique o **client** para o tipo específico de SQL em que você está interessado.

```js
const pg = require('knex')({client: 'pg'});

knex('table')
  .insert({a: 'b'})
  .returning('*')
  .toString();
// "insert into "table" ("a") values ('b')"

pg('table')
  .insert({a: 'b'})
  .returning('*')
  .toString();
// "insert into "table" ("a") values ('b') returning *"
```

##### WIthUserParams

:arrow_right:Método que cria uma instância do Knex com as mesmas conexões, com parâmetros personalizados (por exemplo, para executar as mesmas migrações com parâmetros diferentes).

```js
const knex = require('knex')({
  // Params
});

const knexWithParams = knex.withUserParams({ 
  customUserParam: 'table1'
});
const customUserParam = knexWithParams
  .userParams
  .customUserParam;
```

##### Debug

:arrow_right: passando o atributo "**debug : true**", o objeto de inicialização ativará a depuração para todas as consultas.

##### AsyncStackTraces

:arrow_right: passando o atributo "**asyncStackTraces : true**", o objeto de inicialização ativará a captura de rastreamento de pilha para todos os construtores de consultas, consultas brutas e construtores de esquema.

##### Pool

:arrow_right: O **client** criado pela configuração inicializa um **pool** de conexões utilizando a biblioteca **TarnJS.**

Esse **pool** tem uma configuração padrão de **min: 1**, **max: 10** para as bibliotecas **MySQL** e **PG** e uma única conexão para **sqlite3**.

> **Obs**: Recomenda-se definir o min: 0 para que todas a conexões ociosas possam ser encerradas.

```js
const knex = require('knex')({
  client: 'mysql',
  connection: {
    host : '127.0.0.1',
    port : 3306,
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  },
  pool: { min: 0, max: 7 }
});
```

Para desmontar explicitamente o pool de conexões, poderá usar **knex.destroy([callback])**.

Você pode usar **knex.destroy** passando um retorno de chamada ou encadeando como uma **promise**, mas não ambos.

Para inicial manualmente um pool de conexão destruído :arrow_right: **knex.initialize([config])**.



#### afterCreate

:arrow_right: O callback é chamado quando a pool adquire uma nova conexão do servidor com o BD.

O retorno deve ser chamado para que o **knex** decida se a conexão está ok ou deve ser descartada imediatamente.

```js
const knex = require('knex')({
  client: 'pg',
  connection: {/*...*/},
  pool: {
    afterCreate: function (conn, done) {
      // neste exemplo usamos o API de conexão do driver pg
      conn.query('SET timezone="UTC";', function (err) {
        if (err) {
          // A primeira consulta falhou,
          // retornar erro e não tente fazer a próxima consulta
          done(err, conn);
        } else {
          // fazer a segunda consulta...
          conn.query(
            'SELECT set_limit(0.01);', 
            function (err) {
              // se o erro não for falso, 
              // a conexão é descartada da pool
              // se a conexão aquire foi acionada por uma consulta
              // o erro é passado para próxima query promise
              done(err, conn);
            });
        }
      });
    }
  }
});
```



#### AcquireConnectionTimeout

:arrow_right: Usado para determinar quanto tempo o knex deve esperar antes de lançar um erro.

> O padrão é 60000 ms.

```js
const knex = require('knex')({
  client: 'pg',
  connection: {/*...*/},
  pool: {/*...*/},
  acquireConnectionTimeout: 10000
});
```



#### FetchAsString

Utilizado pelo **Oracledb**. Uma matriz de tipos. Os tipos válidos são '**DATE**', '**NUMBER**' e '**CLOB**'. Quando qualquer coluna com um dos tipos especificados é consultada, os dados da coluna são retornados como uma string em vez da representação padrão.

```js
const knex = require('knex')({
  client: 'oracledb',
  connection: {/*...*/},
  fetchAsString: [ 'number', 'clob' ]
});
```



#### Migrations

Por conveniência, qualquer configuração de migração pode ser especificada ao iniciar a biblioteca.

```js
const knex = require('knex')({
  client: 'mysql',
  connection: {
    host : '127.0.0.1',
    port : 3306,
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  },
  migrations: {
    tableName: 'migrations'
  }
});
```



#### postProcessResponse

:arrow_right:Modificar as linhas retornadas, antes de passá-las para o usuário.

Pode-se fazer, por exemplo, snake_case -> camelCase.

```js
const knex = require('knex')({
  client: 'mysql',
  //  snake_case -> camelCase 
  postProcessResponse: (result, queryContext) => {
    // Fazer: Adicionar "special case" para o "row result". 
    if (Array.isArray(result)) {
      return result.map(row => convertToCamel(row));
    } else {
      return convertToCamel(result);
    }
  }
});
```



#### WrapIdentifier

Com o wrapIdentifier pode-se substituir a forma como os identificadores são transformados. Ele pode ser usado para substituir a funcionalidade padrão e, por exemplo, ajudar a fazer a conversão camelCase -> snake_case:

```js
const knex = require('knex')({
  client: 'mysql',
  // camelCase -> snake_case
  wrapIdentifier: (
    value, 
    origImpl, 
    queryContext
  ) => origImpl(convertToSnakeCase(value))
});
```



#### log

O **Knex** contém algumas funções de log interno para impressão de avisos, erros, informações...

Essas funções de log são registradas no console.log, mas podem ser substituídas usando a opção de log e fornecendo funções alterativas.

```js
const knex = require('knex')({
  log: {
    warn(message) {
    },
    error(message) {
    },
    deprecate(message) {
    },
    debug(message) {
    },
  }
});
```



### Exemplo de uso com Typescript 

```ts
import { Knex } from 'knex';

declare module 'knex/types/tables' {
  interface User {
    id: number;
    name: string;
    created_at: string;
    updated_at: string;
  }
  
  interface Tables {
    // Isto é o mesmo que especificar `knex<User>('users')`
    users: User;
    // Para tipos mais avançados, você pode especificar tipos separados
    // para modelo básico, tipo "insert" e tipo "update".
    // Mas primeiro: observe que se você optar por usar isto, 
    // a digitação básica mostrada acima pode ser ignorada.
    // Então, isto é como especificar
    // knex
    // .insert<{ name: string }>({ name: 'name' })
    // .into<{ name: string, id: number }>('usuários')
    users_composite: Knex.CompositeTableType<

      User,
      Pick<User, 'name'> & Partial<Pick<User, 'created_at' | 'updated_at'>>,
      // Esta interface é utilizada para chamadas de "update()".
      // Em oposição à interface de especificação regular apenas uma vez,
      // ao especificar uma interface de atualização separada, o usuário será
      // necessário para corresponder exatamente. Portanto, é recomendado
      // fornecer interfaces parciais para "update". A menos que você queira sempre
      // requerer algum campo (por exemplo, `Parcial<User> & {updated_at: string }``
      // permitirá a atualização de qualquer campo para o Usuário, mas requer 				  // "updated_at" para ser
      // sempre fornecido também.
      // 
      // Por exemplo, isto permitirá atualizar todos os campos exceto "id".
      // "id" ainda será utilizável para as cláusulas "where" assim
      // knex('users_composite')
      // .update({ name: 'name2' })
      // .where('id', 10)`
      // ainda vai funcionar.
      // Predefinições do tipo "insert" parcial
      Partial<Omit<User, 'id'>>
    >;
  }
}
```





<hr>



# Construtor de consultas Knex 

<a href="https://knexjs.org/guide/query-builder.html#knex-query-builder">Ver Documentação</a>

O coração da biblioteca, o construtor de consultas knex é uma interface usada para construir e executar SQL padrão, como `select`, `insert`, `update`, `delete`.

É necessário apenas o  `tableName.columnName`, `tableName`ou `columnName`, mas em muitos casos também é necessário passar um alias de como esse identificador é referido posteriormente na consulta.

Há duas maneiras de declarar um alias para identificador. Pode-se dar diretamente o sufixo para o identificador (por exemplo `identifierName as aliasName`) ou pode se passar um objeto `{ aliasName: 'identifierName' }`.

```js
knex({ a: 'table', b: 'table' })
  .select({
    aTitle: 'a.title',
    bTitle: 'b.title'
  })
  .whereRaw('?? = ??', ['a.column_1', 'b.column_2'])
```



### Common

**Sintaxe**

>**knex(tableName, options={only: boolean})**
>**knex.[methodName]**

<mark>Obs: Suportado apenas por Postgres</mark>

:arrow_right: **Uso com Typescript**

```js
interface User {
  id: number;
  name: string;
  age: number;
}

knex('users')
  .where('id')
  .first(); 

knex<User>('users') // O usuário é o tipo de linha no banco de dados
  .where('id', 1) // Sua IDE será capaz de ajudar com a conclusão do id
  .first(); // Resolve ao usuário | undefined
```

```js
//Exemplo com JS

/**
 * @typedef {Object} User
 * @property {number} id
 * @property {number} age
 * @property {string} name
 *
 * @returns {Knex.QueryBuilder<User, {}>}
 */
const Users = () => knex('Users')

// A propriedade 'id' pode ser autocompletada pelo editor
Users().where('id', 1) 
```

A maioria das **APIs** **knex** alteram o objeto atual e o retornam. Esse padrão não funciona bem com inferência de tipo.

```ts
knex<User>('users')
  .select('id')
  .then((users) => { 
    // Fazer algo com os usuários
  });

knex<User>('users')
  .select('id')
  .select('age')
  .then((users) => { 
    // Fazer algo com os usuários
  });

// O tipo de usersQueryBuilder é determinado aqui
const usersQueryBuilder = knex<User>('users').select('id');

if (someCondition) {
  // Esta seleção não mudará o tipo de usersQueryBuilder
  usersQueryBuilder.select('age');
}
usersQueryBuilder.then((users) => {
  //...
});

// You can specify the type of result explicitly through a second type parameter:
const queryBuilder = knex<User, Pick<User, "id" | "age">>('users');

// Você pode especificar o tipo de resultado explicitamente através de um segundo 			parâmetro do tipo selecionado

// Portanto, isto irá compilar:
queryBuilder.select('name').then((users) => {
  // O tipo será <User, "id"> mas só terá o nome
})
```

#### Timeout

:arrow_right: Usado para lançarmos um **TimeoutError** se o tempo limite for excedido.

**sintaxe**

> **.timeout(ms, options={cancel: boolean})**

<mark>Suportado em MySQL e PostgreSQL</mark>

```js
knex.select()
  .from('books')
  .timeout(1000)

knex.select()
  .from('books')
  .timeout(1000, { 
    cancel: true // MySQL e PostgreSQL somente
  }) 
```

#### Select

**sintaxe**

> **.select([\*columns])**

```js
knex.select('title', 'author', 'year')
  .from('books')

knex.select()
  .table('books')
```

Obs: Se informar * retorna todos os dados da tabela.

**Exemplo de uso com TS**

```ts
knex.select('id')
  .from<User>('users'); // Tipagem <User, "id">[]

knex.select('users.id')
  .from<User>('users'); // Retorno any[]
//  O TypeScript não nos fornece uma maneira de olhar para uma string e inferir o tipo
//  de uma substring, então voltamos ao any

// Podemos contornar isso usando o knex.ref:
knex.select(knex.ref('id').withSchema('users'))
  .from<User>('users'); // Tipo <User, "id">[]

knex.select('id as identifier')
  .from<User>('users'); // resolve o any[], pela mesma razão que acima;

// As Refs também são úteis aqui:
knex.select(knex.ref('id').as('identifier'))
  .from<User>('users'); // Resolve para { identifier: number; }[]
```



#### AS

**sintaxe**

> .as(nome)

:arrow_right:Permite o alias de uma **subconsulta**, pegando a string que você deseja nomear a consulta atual. Se a consulta não for uma **subconsulta**, ela será ignorada.

```ts
knex.avg('sum_column1')
  .from(function() {
    this.sum('column1 as sum_column1')
      .from('t1')
      .groupBy('column1')
      .as('t1')
  })
  .as('ignored_alias')
```



#### Column

**sintaxe**

> **.column(colunas)**

```js
knex.column('title', 'author', 'year')
  .select()
  .from('books')

knex.column(['title', 'author', 'year'])
  .select()
  .from('books')

knex.column('title', { by: 'author' }, 'year')
  .select()
  .from('books')
```

:arrow_right:Deste modo, defino especificamente as colunas a serem selecionadas em uma consulta de seleção.



#### FROM

**sintaxe**

> **.from([tableName], options={only: boolean})**

<mark>Suportado por PostgreSQL</mark>

```js
knex.select('*')
  .from('users')
```

**Exemplo de uso com Typescript**

```ts
knex.select('id')
  .from('users'); // any[]

knex.select('id')
  .from<User>('users'); // Pick<User, "id">[]
```



#### WITH

**sintaxe**

> **.with(alias, [columns], callback|builder|raw)**

Adicione uma cláusula "**with**" à consulta. As cláusulas "**With**" são suportadas pelo **PostgreSQL**, **Oracle**, **SQLite3** e **MSSQL**. Uma lista de colunas opcional pode ser fornecida após o alias; se fornecido, deve incluir pelo menos um nome de coluna.

```js
knex
  .with(
    'with_alias', 
    knex.raw(
      'select * from "books" where "author" = ?', 
      'Test'
    )
  )
  .select('*')
  .from('with_alias')

knex
  .with(
    'with_alias', 
    ["title"], 
    knex.raw(
      'select "title" from "books" where "author" = ?', 
      'Test'
    )
  )
  .select('*')
  .from('with_alias')

knex
  .with('with_alias', (qb) => {
    qb.select('*')
      .from('books')
      .where('author', 'Test')
  })
  .select('*')
  .from('with_alias')
```



#### UNION

**sintaxe**

> **.union([\*queries], [wrap])**

```js
knex.select('*')
  .from('users')
  .whereNull('last_name')
  .union(function() {
    this.select('*')
      .from('users')
      .whereNull('first_name')
  })

knex.select('*')
  .from('users')
  .whereNull('last_name')
  .union([
    knex.select('*')
      .from('users')
      .whereNull('first_name')
  ])

knex.select('*')
  .from('users')
  .whereNull('last_name')
  .union(
    knex.raw(
      'select * from users where first_name is null'
    ),
    knex.raw(
      'select * from users where email is null'
    )
  )
```

:arrow_right: Cria uma consulta de união, usando uma matriz ou uma lista de retornos de chamada.



#### INSERT

**sintaxe**

> **.insert(dados, [retornando], [opções])**

```js
// Retorna [1] no "mysql", "sqlite", "oracle"; 
// [] no "postgresql" 
//  a menos que o parâmetro "retorno" esteja definido.
knex('books').insert({title: 'Slaughterhouse Five'})

// Normaliza para chaves vazias em inserts com várias linhas:
knex('coords').insert([{x: 20}, {y: 30},  {x: 10, y: 20}])

// Retorna [2] no "mysql", "sqlite"; [2, 3] no "postgresql"
knex
  .insert(
    [
      { title: 'Great Gatsby' }, 
      { title: 'Fahrenheit 451' }
    ], 
    ['id']
  )
  .into('books')
```

Para MSSQL, gatilhos em tabelas podem interromper o retorno de um valor válido das instruções de inserção padrão. Você pode adicionar a `includeTriggerModifications`opção para contornar esse problema. Isso modifica o SQL para que os valores apropriados possam ser retornados. Isso só modifica a instrução se você estiver usando MSSQL, um valor de retorno for especificado e a `includeTriggerModifications`opção estiver definida.

```js
// Add a opção includeTriggerModifications 
// permite que você execute declarações em tabelas  
// que contenham triggers. Só afeta o MSSQL.
knex('books')
  .insert(
    {title: 'Alice in Wonderland'}, 
    ['id'], 
    { includeTriggerModifications: true }
  )
```

Se preferir que as chaves indefinidas sejam substituídas por `NULL`uma `DEFAULT`, pode-se fornecer `useNullAsDefault`o parâmetro de configuração na configuração do **knex**.

```js
const knex = require('knex')({
  client: 'mysql',
  connection: {
    host : '127.0.0.1',
    port : 3306,
    user : 'your_database_user',
    password : 'your_database_password',
    database : 'myapp_test'
  },
  useNullAsDefault: true
});

knex('coords').insert([{x: 20}, {y: 30}, {x: 10, y: 20}])
```

```js
insert into `coords` (`x`, `y`) values (20, NULL), (NULL, 30), (10, 20)"
```



#### UPDATE

**sintaxe**

> **.update(dados, [retornando], [opções])** **.update(chave, valor, [retornando], [opções])**

```js
knex('books')
  .where('published_date', '<', 2000)
  .update({
    status: 'archived',
    thisKeyIsSkipped: undefined
  })

// Retorna [1] no "mysql", "sqlite", "oracle"; 
// [] no "postgresql" 
// a menos que o parâmetro "retorno" esteja definido.
knex('books').update('title', 'Slaughterhouse Five')

/** Retorna 
 * [{ 
 *   id: 42, 
 *   title: "The Hitchhiker's Guide to the Galaxy" 
 * }] **/
knex('books')
  .where({ id: 42 })
  .update({ 
    title: "The Hitchhiker's Guide to the Galaxy" 
  }, ['id', 'title'])
```

:arrow_right: Para MSSQL, gatilhos em tabelas podem interromper o retorno de um valor válido das instruções de atualização padrão. Você pode adicionar a `includeTriggerModifications` para contornar esse problema. Isso modifica o SQL para que os valores apropriados possam ser retornados. Isso só modifica a instrução se você estiver usando MSSQL, um valor de retorno for especificado e a `includeTriggerModifications` estiver definida.

```js
// Adicionando a opção includeTriggerModifications permite que você
// rode declarações em tabelas que contenham triggers.
// somente no MSSQL.
knex('books')
  .update(
    {title: 'Alice in Wonderland'}, 
    ['id', 'title'], 
    { includeTriggerModifications: true }
  )
```



#### DELETE 

**sintaxe**

> **.del([retornando], [opções])**

:arrow_right: Esse método exclui uma ou mais linhas, com base em outras condições especificadas na consulta.

```js
knex('accounts')
  .where('activated', false)
  .del()
```

:arrow_right:Para MSSQL, gatilhos em tabelas podem interromper o retorno de um valor válido das instruções de exclusão padrão. Você pode adicionar a `includeTriggerModifications` para contornar esse problema. Isso modifica o SQL para que os valores apropriados possam ser retornados. Isso só modifica a instrução se você estiver usando MSSQL, um valor de retorno for especificado e a `includeTriggerModifications` estiver definida.

```js
// Adicionando a opção includeTriggerModifications permite que você
// rode declarações em tabelas que contenham triggers. 
// Somente no MSSQL.
knex('books')
  .where('title', 'Alice in Wonderland')
  .del(
    ['id', 'title'], 
    { includeTriggerModifications: true }
  )
```

:arrow_right: Para o **PostgreSQL**, a instrução **Delete** com junções é suportada com a sintaxe clássica de '**join**' e com a sintaxe '**using**'.

```js
knex('accounts')
  .where('activated', false)
  .join('accounts', 'accounts.id', 'users.account_id')
  .del()
```



#### Count

:arrow_right: Executa uma contagem na coluna ou matriz de colunas especificada.

**sintaxe**

> **.count(column|columns|raw, [options])**

```js
knex('users').count('active')

knex('users').count('active', {as: 'a'})

knex('users').count('active as a')

knex('users').count({ a: 'active' })

knex('users').count({ a: 'active', v: 'valid' })

knex('users').count('id', 'active')

knex('users').count({ count: ['id', 'active'] })

knex('users').count(knex.raw('??', ['active']))
```

**Exemplo com TS**

```ts
 knex('users').count('age') // Resolve para: Record<string, number | string>
 knex('users').count({count: '*'}) // Resolve para { count?: string | number | undefined; }
```

Use **countDistinct** para adicionar uma expressão distinta dentro da função de agregação.

```js
knex('users').countDistinct('active')
```



#### Min

**sintaxe**

> **.min(coluna|colunas|bruto, [opções])**

:arrow_right: Obtém o valor mínimo para a coluna ou matriz de colunas especificada.

```js
knex('users').min('age')

knex('users').min('age', {as: 'a'})

knex('users').min('age as a')

knex('users').min({ a: 'age' })

knex('users').min({ a: 'age', b: 'experience' })

knex('users').min('age', 'logins')

knex('users').min({ min: ['age', 'logins'] })

knex('users').min(knex.raw('??', ['age']))
```



#### Max

**sintaxe**

> **.max(coluna|colunas|bruto, [opções])**

:arrow_right: Obtém o valor máximo para a coluna ou matriz de colunas especificada.

```js
knex('users').max('age')

knex('users').max('age', {as: 'a'})

knex('users').max('age as a')

knex('users').max({ a: 'age' })

knex('users').max('age', 'logins')

knex('users').max({ max: ['age', 'logins'] })

knex('users').max({ max: 'age', exp: 'experience' })

knex('users').max(knex.raw('??', ['age']))
```



#### SOMA

**sintaxe**

> **.sum(coluna|colunas|bruto)**

:arrow_right: Recupere a soma dos valores de uma determinada coluna ou matriz de colunas

```js
knex('users').sum('products')

knex('users').sum('products as p')

knex('users').sum({ p: 'products' })

knex('users').sum('products', 'orders')

knex('users').sum({ sum: ['products', 'orders'] })

knex('users').sum(knex.raw('??', ['products']))
```

Use **sumDistinct** para adicionar uma expressão distinta dentro da função de agregação.

```js
knex('users').sumDistinct('products')
```



Mais exemplos:

<a href="https://knexjs.org/guide/query-builder.html#rank">Documentação</a>



<hr>



## Where Clauses

Existem vários métodos para auxiliar em cláusulas **where** dinâmicas. Em muitos lugares, funções podem ser usadas no lugar de valores, construindo **subconsultas**. Na maioria dos lugares, consultas knex existentes podem ser usadas para compor **subconsultas**, etc. 

### where

**sintaxe**

>**.where(~mixed~)**
>**.orWhere**

```js
knex('users').where({
  first_name: 'Test',
  last_name:  'User'
}).select('id')
```

chave, valor:

```js
knex('users').where('id', 1)
```

Funções:

```js
knex('users')
  .where((builder) =>
    builder
      .whereIn('id', [1, 11, 15])
      .whereNotIn('id', [17, 19])
  )
  .andWhere(function() {
    this.where('id', '>', 10)
  })
```

Cadeia Agrupada:

```js
knex('users').where(function() {
  this.where('id', 1).orWhere('id', '>', 10)
}).orWhere({name: 'Tester'})
```

Operadores:

```js
knex('users').where('columnName', 'like', '%rowlikeme%')
```



**Exemplo de uso**

```js
knex('users').where('votes', '>', 100)

const subquery = knex('users')
  .where('votes', '>', 100)
  .andWhere('status', 'active')
  .orWhere('name', 'John')
  .select('id');

knex('accounts').where('id', 'in', subquery)
```



### Where NOT

**sintaxe**

> **.whereNot(~mixed~)** **.orWhereNot**

```js
knex('users').whereNot({
  first_name: 'Test',
  last_name:  'User'
}).select('id')
```

Valor, Chave:

```js
knex('users').whereNot('id', 1)
```

Cadeia Agrupada:

```js
knex('users').whereNot(function() {
  this.where('id', 1).orWhereNot('id', '>', 10)
}).orWhereNot({name: 'Tester'})
```

Operador:

```js
knex('users').whereNot('votes', '>', 100)
```

<mark>**WhereNot** não é adequado para **subconsultas** do tipo "**in**" e "**between**". Você deve usar "**not in**" e "**not between**" em vez disso.</mark>



### WhereLike

**sintaxe**

>**.whereLike(coluna, string|builder|raw)**
>**.orWhereLike**

:arrow_right: Adiciona uma cláusula where com comparação de **substring** que **diferencia maiúsculas de minúsculas** em uma determinada coluna com um determinado valor.

```js
knex('users').whereLike('email', '%mail%')

knex('users')
  .whereLike('email', '%mail%')
  .andWhereLike('email', '%.com')
  .orWhereLike('email', '%name%')
```



### WhereILike

>**.whereILike(column, string|builder|raw)**
>**.orWhereILike**

:arrow_right: Adiciona uma cláusula where com comparação de **substring** que **não diferencia maiúsculas de minúsculas** em uma determinada coluna com um determinado valor.

```js
knex('users').whereILike('email', '%mail%')

knex('users')
  .whereILike('email', '%MAIL%')
  .andWhereILike('email', '%.COM')
  .orWhereILike('email', '%NAME%')
```



:heavy_plus_sign: Mais "**Where Clauses**" <a href="https://knexjs.org/guide/query-builder.html#where-clauses">Aqui</a>



<hr>



## JOIN METHODS

<a href="https://knexjs.org/guide/query-builder.html#wherejsonsubsetof">Documentação</a>

### Join

> **.join(table, first, [operator], second)**

:arrow_right:  Pode ser usado para especificar junções entre tabelas, com o primeiro argumento sendo a tabela de junção, os próximos três argumentos sendo a primeira coluna de junção, o operador de junção e a segunda coluna de junção, respectivamente.

```js
knex('users')
  .join('contacts', 'users.id', '=', 'contacts.user_id')
  .select('users.id', 'contacts.phone')

knex('users')
  .join('contacts', 'users.id', 'contacts.user_id')
  .select('users.id', 'contacts.phone')
```

Para junções agrupadas, especifique uma função como o segundo argumento para a consulta de junção e use `on`com `orOn`ou `andOn`para criar junções agrupadas com parênteses.

```js
knex.select('*').from('users').join('accounts', function() {
  this
    .on('accounts.id', '=', 'users.account_id')
    .orOn('accounts.owner_id', '=', 'users.id')
})
```

Para instruções de junção aninhadas, especifique uma função como primeiro argumento `on`de `orOn`ou`andOn`

```js
knex.select('*').from('users').join('accounts', function() {
  this.on(function() {
    this.on('accounts.id', '=', 'users.account_id')
    this.orOn('accounts.owner_id', '=', 'users.id')
  })
})
```

Também é possível usar um objeto para representar a sintaxe de junção.

```js
knex.select('*')
  .from('users')
  .join('accounts', {'accounts.id': 'users.account_id'})
```

Se você precisar usar um valor literal (string, número ou booleano) em uma junção em vez de uma coluna, use `knex.raw`.

```js
knex.select('*')
  .from('users')
  .join(
    'accounts', 
    'accounts.type',
    knex.raw('?', ['admin'])
  )
```



### LeftJoin

**sintaxe**

> **.leftJoin(table, ~mixed~)**

```js
knex.select('*')
  .from('users')
  .leftJoin('accounts', 'users.id', 'accounts.user_id')

knex.select('*')
  .from('users')
  .leftJoin('accounts', function() {
    this
      .on('accounts.id', '=', 'users.account_id')
      .orOn('accounts.owner_id', '=', 'users.id')
  })
```



### RightJoin

**sintaxe**

> **.rightJoin(table, ~mixed~)**

```js
knex.select('*')
  .from('users')
  .rightJoin('accounts', 'users.id', 'accounts.user_id')

knex.select('*')
  .from('users')
  .rightJoin('accounts', function() {
    this
      .on('accounts.id', '=', 'users.account_id')
      .orOn('accounts.owner_id', '=', 'users.id')
  })
```



### CrossJoin

> **.crossJoin(table, ~mixed~)**

<mark>As condições de junção cruzada são suportadas apenas no MySQL e SQLite3. Para condições de junção, use innerJoin.</mark>

```js
knex.select('*')
  .from('users')
  .crossJoin('accounts')

knex.select('*')
  .from('users')
  .crossJoin('accounts', 'users.id', 'accounts.user_id')

knex.select('*')
  .from('users')
  .crossJoin('accounts', function() {
    this
      .on('accounts.id', '=', 'users.account_id')
      .orOn('accounts.owner_id', '=', 'users.id')
  })
```



<hr>



## OnClauses

### OnIn

**sintaxe**

> **.onIn(column, values)**

```js
knex.select('*')
  .from('users')
  .join('contacts', function() {
    this
      .on('users.id', '=', 'contacts.id')
      .onIn('contacts.id', [7, 15, 23, 41])
  })
```

### OnNotIn

**sintaxe**

> **.onNotIn(column, values)**

:arrow_right: Adiciona uma cláusula onNotIn à consulta.

```js
knex.select('*')
  .from('users')
  .join('contacts', function() {
    this
      .on('users.id', '=', 'contacts.id')
      .onNotIn('contacts.id', [7, 15, 23, 41])
  })
```

### OnExists

**sintaxe**

> **.onExists(construtor | retorno de chamada)**

```js
knex.select('*').from('users').join('contacts', function() {
  this
    .on('users.id', '=', 'contacts.id')
    .onExists(function() {
      this.select('*')
        .from('accounts')
        .whereRaw('users.account_id = accounts.id');
    })
})
```



### OnNotExists

**sintaxe**

> **.onNotExists(builder | callback)**

```js
knex.select('*').from('users').join('contacts', function() {
  this
    .on('users.id', '=', 'contacts.id')
    .onNotExists(function() {
      this.select('*')
        .from('accounts')
        .whereRaw('users.account_id = accounts.id');
    })
})
```



### onBetween

**sintaxe**

> **.onBetween(column, range)**

```js
knex.select('*').from('users').join('contacts', function() {
  this
    .on('users.id', '=', 'contacts.id')
    .onBetween('contacts.id', [5, 30])
})
```



### OnNotBetween

**Sintaxe**

> **.onNotBetween(column, range)**

```js
knex.select('*').from('users').join('contacts', function() {
  this
    .on('users.id', '=', 'contacts.id')
    .onNotBetween('contacts.id', [5, 30])
})
```



<hr>



## ClearClauses

<a href="https://knexjs.org/guide/query-builder.html#clearclauses">Documentação</a>

### Group By

:arrow_right:  **Adds a group by clause to the query.**

```js
knex('users').groupBy('count')
```



### OrderBy

**sintaxe**

> **.orderBy(column|columns, [direction], [nulls])**

:arrow_right: Simples

```js
knex('users').orderBy('email')

knex('users').orderBy('name', 'desc')

knex('users').orderBy('name', 'desc', 'first')
```

:arrow_right: Multiplas Colunas:

```js
knex('users').orderBy([
  'email', { column: 'age', order: 'desc' }
])

knex('users').orderBy([
  { column: 'email' }, 
  { column: 'age', order: 'desc' }
])

knex('users').orderBy([
  { column: 'email' }, 
  { column: 'age', order: 'desc', nulls: 'last' }
])
```



<hr>



## Having Clauses

### having

**sintaxe**

> **.having(column, operator, value)**

```js
knex('users')
  .groupBy('count')
  .orderBy('name', 'desc')
  .having('count', '>', 100)
```

### havingIn

**sintaxe**

> **.havingIn(column, values)**

```js
knex.select('*')
  .from('users')
  .havingIn('id', [5, 3, 10, 17])
```



### havingNotIn

**sintaxe**

> **.havingNotIn(column, values)**

```js
knex.select('*')
  .from('users')
  .havingNotIn('id', [5, 3, 10, 17])
```



##### Mais exemplos: <a href="https://knexjs.org/guide/query-builder.html#having-clauses">Leia a documentação</a>



<hr>



## Schema Builder

<a href="https://knexjs.org/guide/schema-builder.html">Documentação completa</a>

### WithSchema

**sintaxe**

> **knex.schema.withSchema([schemaName])**

```js
knex.schema.withSchema('public').createTable('users', function (table) {
  table.increments();
})
```



### CreateTable

>**knex.schema.createTable(tableName, callback)**

```js
knex.schema.createTable('users', function (table) {
  table.increments();
  table.string('name');
  table.timestamps();
})
```



### CreateTableLike

:arrow_right:Criar Tabela com base em uma tabela Existente:

**sintaxe**

> **knex.schema.createTableLike(tableName, tableNameToCopy, [callback])**

```js
knex.schema.createTableLike('new_users', 'users')

// a tabela "new_users" que contem coluna 
// de usuários e duas novas colunas 'age' e 'last_name'.
knex.schema.createTableLike('new_users', 'users', (table) => {
  table.integer('age');
  table.string('last_name');
})
```



### DropTable

**sintaxe**

>**knex.schema.dropTable(tableName)**

```js
knex.schema.dropTable('users')
```



### DropTableIfExists

:arrow_right: **Excluir a tabela se ela existir**

> **knex.schema.dropTableIfExists(tableName)**

```js
knex.schema.dropTableIfExists('users')
```



### renameTable

**sintaxe**

> **knex.schema.renameTable(from, to)**

```js
knex.schema.renameTable('users', 'old_users')
```



### hasTable

:arrow_right: **Verificar se existe a tabela** 

**sintaxe**

> **knex.schema.hasTable(tableName)**

```js
knex.schema.hasTable('users').then(function(exists) {
  if (!exists) {
    return knex.schema.createTable('users', function(t) {
      t.increments('id').primary();
      t.string('first_name', 100);
      t.string('last_name', 100);
      t.text('bio');
    });
  }
});
```



### Table

:arrow_right: Escolhe uma tabela de banco de dados e, em seguida, modifica a tabela, usando as funções **Schema** **Building** dentro do retorno de chamada.

**sintaxe**

> **knex.schema.table(tableName, callback)**

```js
knex.schema.table('users', function (table) {
  table.dropColumn('name');
  table.string('first_name');
  table.string('last_name');
})
```



### AlterTable

:arrow_right: **Alternar Tabela**

**sintaxe**

> **knex.schema.alterTable(tableName, callback)**

```js
knex.schema.alterTable('users', function (table) {
  table.dropColumn('name');
  table.string('first_name');
  table.string('last_name');
})
```



<a href="https://knexjs.org/guide/schema-builder.html#createview">Ver mais na Documentação</a>



<hr>

## Schema Building

<a href="https://knexjs.org/guide/schema-builder.html#schema-building">Documentação completa</a>

### Date

**sintaxe**

> **table.date(name)**



### DateTime

**sintaxe**

> **table.datetime(name, options={[useTz: boolean], [precision: number]})**

:arrow_right: Adiciona uma coluna de data e hora. Por padrão o PostgreSQL cria coluna com fuso horário (tipo timestamptz). Esse comportamento pode ser substituído passando a opção useTz (que por padrão é verdadeira para o PostgreSQL). MySQL e MSSQL não possuem a opção useTz.

**Uma opção de precisão pode ser passada:**

```js
table.datetime('some_time', { precision: 6 }).defaultTo(knex.fn.now(6))
```



### Timestamp

**sintaxe**

> **table.timestamp(name, options={[useTz: boolean], [precision: number]})**

```js
table.timestamp('created_at').defaultTo(knex.fn.now());
```

:arrow_right: No PostgreSQL e MySQL uma opção de precisão pode ser passada:

```js
table.timestamp('created_at', { precision: 6 }).defaultTo(knex.fn.now(6));
```

:arrow_right: No PostgreSQL e MSSQL uma opção de fuso horário pode ser passada:

```js
table.timestamp('created_at', { useTz: true });
```



### UUID

**sintaxe**

> **table.uuid(name, options=({[useBinaryUuid:boolean],[primaryKey:boolean]})**

:arrow_right: Adiciona uma coluna **uuid** - isso usa o tipo **uuid** embutido no **PostgreSQL** e retorna para um char(36) em outros bancos de dados por padrão. 



### Primary

**sintaxe**

> **table.primary(columns, options=({[constraintName:string],[deferrable:'not deferrable'|'deferred'|'immediate']})**

:arrow_right: Crie uma restrição de chave primária na tabela usando input `columns`

```js
knex.schema.alterTable('users', function(t) {
  t.unique('email')
})
knex.schema.alterTable('job', function(t) {
  t.primary('email',{constraintName:'users_primary_key',deferrable:'deferred'})
})
```



## Foreign

**sintaxe**

> **table.foreign(columns, [foreignKeyName])[.onDelete(statement).onUpdate(statement).withKeyName(foreignKeyName).deferrable(type)]**

:arrow_right: Adiciona uma restrição de chave estrangeira a uma tabela para uma coluna existente usando `table.foreign(column).references(column)`ou várias colunas usando `table.foreign(columns).references(columns).inTable(table)`.

```js
knex.schema.table('users', function (table) {
  table.integer('user_id').unsigned()
  table.foreign('user_id').references('Items.user_id_in_items').deferrable('deferred')
})
```



<a href="https://knexjs.org/guide/schema-builder.html#chainable-methods">Ver mais na documentação</a>



<hr>



# Migrations

<a href="https://knexjs.org/guide/migrations.html">Documentação completa</a>

:information_source: As migrações permitem que você defina conjuntos de alterações de esquema para que atualizar um banco de dados seja muito fácil.

## CLI de migração

```bash
$ npm install knex -g
```

As migrações usam um **knexfile** , que especifica várias configurações para o módulo. Para criar um novo knexfile, execute o seguinte:

```bash
$ knex init

# or for .ts

$ knex init -x ts
```

:arrow_right: Será gerado um arquivo chamado **knexfile.js** que contém nossas várias configurações de banco de dados. 

**Configuração básica de um knexfile.js**

```sh
module.exports = {
  client: 'pg',
  connection: process.env.DATABASE_URL || { 
    user: 'me', 
    database: 'my_app' 
  }
};
```

Você também pode exportar uma função assíncrona do knexfile. Isso é útil quando você precisa buscar credenciais de um local seguro...

```sh
module.exports = async () => {
  const configuration = await fetchConfiguration();
  return {
    ...configuration,
    migrations: {}
  }
};

```

**Configuração do ambiente**

```sh
module.exports = {
  development: {
    client: 'pg',
    connection: { user: 'me', database: 'my_app' }
  },
  production: { 
    client: 'pg', 
    connection: process.env.DATABASE_URL 
  }
};
```

Depois de ter um knexfile.js, você pode usar a ferramenta de migração para criar arquivos de migração para o diretório especificado (migrações padrão). A criação de novos arquivos de migração pode ser feita executando:

```sh
$ knex migrate:make migration_name 

# or for .ts

$ knex migrate:make migration_name -x ts
```

Depois de terminar de escrever as migrações, você pode atualizar o banco de dados correspondente ao seu `NODE_ENV`executando:

```sh
$ knex migrate:latest
```

**Exemplo de migração gerada**

```sh
module.exports = {
  client: 'pg',
  migrations: {
    extension: 'ts'
  }
};
```

**Para reverter o último lote de migrações:**

```sh
$ knex migrate:rollback
```

**Para reverter todas as migrações concluídas:**

```bash
$ knex migrate:rollback --all
```

**Para executar a próxima migração que ainda não foi executada**

```sh
$ knex migrate:up
```

**Para executar a migração especificada que ainda não foi executada**

```sh
$ knex migrate:up 001_migration_name.js
```

**Para desfazer a última migração que foi executada**

```sh
$ knex migrate:down
```

**Para desfazer a migração especificada que foi executada**

```sh
$ knex migrate:down 001_migration_name.js
```

**Para listar as migrações concluídas e pendentes:**

```sh
$ knex migrate:list
```



### Make

**sintaxe**

> **knex.migrate.make(name, [config])**

:arrow_right: Cria uma nova migração, com o nome da migração que está sendo adicionada.



### Latest

**sintaxe**

> **knex.migrate.latest([config])**

:arrow_right: Executa todas as migrações que ainda não foram executadas.

```js
knex.migrate.latest()
  .then(function() {
    return knex.seed.run();
  })
  .then(function() {
    // as migrações estão concluídas
  });
```



### Custom Migration

```js
//  Criar uma classe de migração personalizada
class MyMigrationSource {
  // Deve retornar uma Promise contendo uma lista de migrações. 
  // As migrações podem ser o que você quiser, 
  // eles serão passados como argumentos para getMigrationName  
  // e getMigration
  getMigrations() {
// Neste exemplo, estamos apenas retornando nomes de migração
    return Promise.resolve(['migration1'])
  }

  getMigrationName(migration) {
    return migration;
  }

  getMigration(migration) {
    switch(migration) {
      case 'migration1':
        return {
          up(knex)   { /* ... */ },
          down(knex) { /* ... */ },
        }
    }
  }
}

// passe uma instância de sua fonte de migração como configuração knex
knex.migrate.latest({ 
  migrationSource: new MyMigrationSource() 
})
```





## Seed Files

Os **Seeders** permitem que você preencha seu banco de dados com dados de teste ou independentemente de seus arquivos de migração.

**Seed CLI**

:arrow_right: Para criar um arquivo **Seed**, execute:

```bash
knex seed:make seed_name
```

Configuração básica do seed:

```js
module.exports = {
  // ...
  development: {
    client: {/* ... */},
    connection: {/* ... */},
    seeds: {
        directory: './seeds/dev'             //<<---Caminhos absolitos não são suportados
    }
  }
  // ...
  }
```

**Para executar arquivos seed, execute:**

```bash
$ knex seed:run
```

Para executar arquivos **seed** **específicos**, execute:

```sh
$ knex seed:run --specific=seed-filename.js --specific=another-seed-filename.js
```



## Transações em Migrações

:arrow_right: Por padrão, cada migração é executada dentro de uma transação. Sempre que necessário, pode-se desabilitar transações para todas as migrações por meio da opção de configuração `config.disableTransactions`de migração comum ou por migração, expondo uma propriedade booleana `config.transaction`de um arquivo de migração:

```js
exports.up = function(knex) {
  return knex.schema
    .createTable('users', function (table) {
        table.increments('id');
        table.string('first_name', 255).notNullable();
        table.string('last_name', 255).notNullable();
    })
    .createTable('products', function (table) {
        table.increments('id');
        table.decimal('price').notNullable();
        table.string('name', 1000).notNullable();
    });
};

exports.down = function(knex) {
  return knex.schema
      .dropTable("products")
      .dropTable("users");
};

exports.config = { transaction: false };
```



Para o knexfile você pode usar uma exportação padrão,
ela terá precedência sobre a exportação nomeada.

```js
/**
 * filename: knexfile.js
 * Para o arquivo knex você pode usar uma exportação padrão
 **/        
export default {
  client: 'sqlite3',
  connection: {
    filename: '../test.sqlite3',
  },
  migrations: {
    directory: './migrations',
  },
  seeds: {
    directory: './seeds',
  },
}

/**
 * filename: knexfile.js
 * Deixe a knex encontrar a configuração, fornecendo exportações nomeadas,
 * mas se for exportado por padrão, ele terá precedência e será usado em seu lugar
 **/
const config = {
  client: 'sqlite3',
  connection: {
    filename: '../test.sqlite3',
  },
  migrations: {
    directory: './migrations',
  },
  seeds: {
    directory: './seeds',
  },
};
/** isto será usado, tem precedência sobre a exportação nomeada */
export default config;
/** Exportações nomeadas, serão usadas se você não forneceu uma exportação padrão */
export const { client, connection, migrations, seeds } = config;
```

Os arquivos de propagação e migração precisam seguir as convenções do Knex

```js
// file: seed.js
/** 
 * O mesmo que com os módulos do CommonJS
 * Você precisará exportar uma função nomeada "seed".
 * */
export function seed(next) {
  // ...a lógica do seed aqui
}

// file: migration.js
/** 
 * O mesmo que a versão CommonJS, o arquivo de migração deve ser exportado 
 * funções : "up" e "down" ;
 */
export function up(knex) {
  // ... lógica da migração aqui
}
export function down(knex) {
// ... Lógica da migração aqui
}
```





<hr>



## Interfaces

:arrow_right: O **Knex.js** oferece várias opções para lidar com a saída da consulta.



### Promises

:arrow_right: O principal benefício das promises é a capacidade de capturar erros lançados sem travar o aplicativo do nó, fazendo com que seu código se comporte como um **.try / .catch / .finally** em código síncrono.

```js
knex.select('name')
  .from('users')
  .where('id', '>', 20)
  .andWhere('id', '<', 200)
  .limit(10)
  .offset(x)
  .then(function(rows) {
    return _.pluck(rows, 'name');
  })
  .then(function(names) {
    return knex.select('id')
      .from('nicknames')
      .whereIn('nickname', names);
  })
  .then(function(rows) {
    console.log(rows);
  })
  .catch(function(error) {
    console.error(error)
  });
```



<a href="https://knexjs.org/guide/interfaces.html#promises">Ver mais na documentação</a>



<hr>



## Transações

:information_source: As transações são um recurso importante dos bancos de dados relacionais, pois permitem a recuperação correta de falhas e mantêm um banco de dados consistente mesmo em casos de falha do sistema. 

As transações são tratadas passando uma função de manipulador para `knex.transaction`. A função handler aceita um único argumento, um objeto que pode ser usado de duas maneiras:

1. Como a conexão knex "consciente da promessa"
2. Como um objeto passado para uma consulta come eventualmente chamar **commit** ou **rollback**.

```js
// Usando trx como um construtor de consultas:
knex.transaction(function(trx) {

  const books = [
    {title: 'Canterbury Tales'},
    {title: 'Moby Dick'},
    {title: 'Hamlet'}
  ];

  return trx
    .insert({name: 'Old Books'}, 'id')
    .into('catalogues')
    .then(function(ids) {
      books.forEach((book) => book.catalogue_id = ids[0]);
      return trx('books').insert(books);
    });
})
.then(function(inserts) {
  console.log(inserts.length + ' new books saved.');
})
.catch(function(error) {
  // Se chegarmos aqui, isso significa que 
  // nem os catálogos dos 'Livros Antigos' inserem,
  // nem qualquer um dos encartes dos livros terá sido realizado.
  console.error(error);
});
```

Outro Exemplo

```js
// Usando trx como um objeto de transação:
knex.transaction(function(trx) {

  const books = [
    {title: 'Canterbury Tales'},
    {title: 'Moby Dick'},
    {title: 'Hamlet'}
  ];

  knex.insert({name: 'Old Books'}, 'id')
    .into('catalogues')
    .transacting(trx)
    .then(function(ids) {
      books.forEach((book) => book.catalogue_id = ids[0]);
      return knex('books').insert(books).transacting(trx);
    })
    .then(trx.commit)
    .catch(trx.rollback);
})
.then(function(inserts) {
  console.log(inserts.length + ' new books saved.');
})
.catch(function(error) {
  // Se chegarmos aqui, isso significa que 
  // nem os catálogos dos 'Old Books' inserem,
  // nem qualquer um dos encartes dos books terá sido realizado.
  console.error(error);
});
```

O Exemplo de cima, usando **async/await.**

```js
try {
  await knex.transaction(async trx => {

    const books = [
      {title: 'Canterbury Tales'},
      {title: 'Moby Dick'},
      {title: 'Hamlet'}
    ];
    
    const ids = await trx('catalogues')
      .insert({
        name: 'Old Books'
      }, 'id')

    books.forEach((book) => book.catalogue_id = ids[0])
    const inserts = await trx('books').insert(books)
    
    console.log(inserts.length + ' new books saved.')
  })
} catch (error) {
  // Se chegarmos aqui, isso significa que nem os catálogos dos 'Old Books' inserem,
  // nem qualquer um dos encartes dos books terá sido realizado.
  console.error(error);
}
```



<a href="https://knexjs.org/guide/transactions.html">Ver mais na documentação</a>

