# Estudo Sobre o KnexJS

```sh
$ npm install knex --save

# Then add one of the following (adding a --save) flag:
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

O próprio módulo **Knex** é uma função que retorna um objeto de configuração para o **Knex**, aceitando alguns parâmetros. O parâmetro **client** é necessário e determina qual adaptador de cliente será usado com a biblioteca:

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

As opções de conexão são passadas diretamente para o **client** de banco de dados apropriado para criar a conexão, e podem ser um objeto, uma cadeia de conexão ou uma função que devolva um objeto.



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
  client: 'sqlite3', // or 'better-sqlite3'
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

Por padrão, o objeto de configuração recebido por meio de uma função é armazenado em **cache** e reutilizado para todas as conexões. Para alterar esse comportamento, uma **expirationChecker** função pode ser retornada como parte do objeto de configuração. O **expirationChecker** é consultado antes de tentar criar novas conexões , e caso retorne **true**, um novo objeto de configuração é recuperado. Por exemplo, para trabalhar com um token de autenticação com vida útil limitada:

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

**Useparams** :arrow_forward: É um parâmetro opcional que permite passar parâmetros arbitrários que serão acessíveis via propriedade **knex.userParams**.

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



Especifique o client para o tipo específico de SQL em que você está interessado.

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
      // in this example we use pg driver's connection API
      conn.query('SET timezone="UTC";', function (err) {
        if (err) {
          // first query failed, 
          // return error and don't try to make next query
          done(err, conn);
        } else {
          // do the second query...
          conn.query(
            'SELECT set_limit(0.01);', 
            function (err) {
              // if err is not falsy, 
              //  connection is discarded from pool
              // if connection aquire was triggered by a 
              // query the error is passed to query promise
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
  // overly simplified snake_case -> camelCase converter
  postProcessResponse: (result, queryContext) => {
    // TODO: add special case for raw results 
    // (depends on dialect)
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
  // overly simplified camelCase -> snake_case converter
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
    // This is same as specifying `knex<User>('users')`
    users: User;
    // For more advanced types, you can specify separate type
    // for base model, "insert" type and "update" type.
    // But first: notice that if you choose to use this, 
    // the basic typing showed above can be ignored.
    // So, this is like specifying
    //    knex
    //    .insert<{ name: string }>({ name: 'name' })
    //    .into<{ name: string, id: number }>('users')
    users_composite: Knex.CompositeTableType<
      // This interface will be used for return type and 
      // `where`, `having` etc where full type is required 
      User,
      // Specifying "insert" type will also make sure
      // data matches interface in full. Meaning
      // if interface is `{ a: string, b: string }`,
      // `insert({ a: '' })` will complain about missing fields.
      // 
      // For example, this will require only "name" field when inserting
      // and make created_at and updated_at optional.
      // And "id" can't be provided at all.
      // Defaults to "base" type.
      Pick<User, 'name'> & Partial<Pick<User, 'created_at' | 'updated_at'>>,
      // This interface is used for "update()" calls.
      // As opposed to regular specifying interface only once,
      // when specifying separate update interface, user will be
      // required to match it  exactly. So it's recommended to
      // provide partial interfaces for "update". Unless you want to always
      // require some field (e.g., `Partial<User> & { updated_at: string }`
      // will allow updating any field for User but require updated_at to be
      // always provided as well.
      // 
      // For example, this wil allow updating all fields except "id".
      // "id" will still be usable for `where` clauses so
      //      knex('users_composite')
      //      .update({ name: 'name2' })
      //      .where('id', 10)`
      // will still work.
      // Defaults to Partial "insert" type
      Partial<Omit<User, 'id'>>
    >;
  }
}
```



# Construtor de consultas Knex 

<a href="https://knexjs.org/guide/query-builder.html#knex-query-builder">Ver Documentação</a>

O coração da biblioteca, o construtor de consultas knex é uma interface usada para construir e executar SQL padrão, como `select`, `insert`, `update`, `delete`.

Mais comumente, é necessário apenas plain `tableName.columnName`, `tableName`ou `columnName`, mas em muitos casos também é necessário passar um alias de como esse identificador é referido posteriormente na consulta.

Há duas maneiras de declarar um alias para identificador. Pode-se dar diretamente `as aliasName`o sufixo para o identificador (por exemplo `identifierName as aliasName`) ou pode se passar um objeto `{ aliasName: 'identifierName' }`.

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
  .first(); // Resolves to any

knex<User>('users') // User is the type of row in database
  .where('id', 1) // Your IDE will be able to help with the completion of id
  .first(); // Resolves to User | undefined
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

// 'id' property can be autocompleted by editor
Users().where('id', 1) 
```

A maioria das APIs knex alteram o objeto atual e o retornam. Esse padrão não funciona bem com inferência de tipo.

```ts
knex<User>('users')
  .select('id')
  .then((users) => { // Type of users is inferred as Pick<User, "id">[]
    // Do something with users
  });

knex<User>('users')
  .select('id')
  .select('age')
  .then((users) => { // Type of users is inferred as Pick<User, "id" | "age">[]
    // Do something with users
  });

// The type of usersQueryBuilder is determined here
const usersQueryBuilder = knex<User>('users').select('id');

if (someCondition) {
  // This select will not change the type of usersQueryBuilder
  // We can not change the type of a pre-declared variable in TypeScript
  usersQueryBuilder.select('age');
}
usersQueryBuilder.then((users) => {
  // Type of users here will be Pick<User, "id">[]
  // which may not be what you expect.
});

// You can specify the type of result explicitly through a second type parameter:
const queryBuilder = knex<User, Pick<User, "id" | "age">>('users');

// But there is no type constraint to ensure that these properties have actually been
// selected.

// So, this will compile:
queryBuilder.select('name').then((users) => {
  // Type of users is Pick<User, "id"> but it will only have name
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
    cancel: true // MySQL and PostgreSQL only
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
  .from<User>('users'); // Resolves to Pick<User, "id">[]

knex.select('users.id')
  .from<User>('users'); // Resolves to any[]
// ^ TypeScript doesn't provide us a way to look into a string and infer the type
//   from a substring, so we fall back to any

// We can side-step this using knex.ref:
knex.select(knex.ref('id').withSchema('users'))
  .from<User>('users'); // Resolves to Pick<User, "id">[]

knex.select('id as identifier')
  .from<User>('users'); // Resolves to any[], for same reason as above

// Refs are handy here too:
knex.select(knex.ref('id').as('identifier'))
  .from<User>('users'); // Resolves to { identifier: number; }[]
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
  .from('users'); // Resolves to any[]

knex.select('id')
  .from<User>('users'); // Results to Pick<User, "id">[]
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
// Returns [1] in "mysql", "sqlite", "oracle"; 
// [] in "postgresql" 
// unless the 'returning' parameter is set.
knex('books').insert({title: 'Slaughterhouse Five'})

// Normalizes for empty keys on multi-row insert:
knex('coords').insert([{x: 20}, {y: 30},  {x: 10, y: 20}])

// Returns [2] in "mysql", "sqlite"; [2, 3] in "postgresql"
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
// Adding the option includeTriggerModifications 
// allows you to run statements on tables 
// that contain triggers. Only affects MSSQL.
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

// Returns [1] in "mysql", "sqlite", "oracle"; 
// [] in "postgresql" 
// unless the 'returning' parameter is set.
knex('books').update('title', 'Slaughterhouse Five')

/** Returns  
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
// Adding the option includeTriggerModifications allows you
// to run statements on tables that contain triggers.
// Only affects MSSQL.
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
// Adding the option includeTriggerModifications allows you
// to run statements on tables that contain triggers. 
// Only affects MSSQL.
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
 knex('users').count('age') // Resolves to: Record<string, number | string>
 knex('users').count({count: '*'}) // Resolves to { count?: string | number | undefined; }
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



