# Template Express Relacional

Este é o template da SofTeam para aplicações Express com banco de dados relacional, servindo como diretriz para os projetos da empresa. Para ver o template com MongoDB, clique [aqui](https://github.com/softeam-org/mongo-express-template).

  1. [Pré-Requisitos](#pré-requisitos)
  2. [Configuração](#configuração)
  3. [Scripts](#scripts)
  4. [Estrutura](#estrutura)
  5. [Criando novos módulos](#criando-novos-módulos)
   6. [Dependências](#dependências)
  
## Pré-requisitos

- [Git](https://git-scm.com/)
- [Node.js](https://nodejs.org/en/)

## Configuração

1. Clone o repositório e instale as dependências.
```sh
# Clonando o repositório
git clone https://github.com/softeam-org/express-template.git

# Navegando para a pasta raiz
cd express-template

# Instalando as dependências do projeto
npm i
```
2. Crie um arquivo chamado `.env` na pasta raiz do projeto e preencha ele tomando como base o arquivo `.env.example`. 

## Scripts

Para executar um script, rode `npm run <nome_do_script>` na pasta raiz do projeto. 
- **build:** Transpila o código de ES6 para CommonJS utilizando o Babel.
- **start:** Roda o script build e executa o código que acabou de ser transpilado.
- **dev:** Transpila o código em tempo de execução, iniciando a aplicação instantaneamente. 
- **test:** Executa os testes.

## Estrutura

### Pastas

- Em `controllers/`, ficam as classes responsáveis por implementar a lógica de negócio, expondo CRUDs e outras funcionalidades.
- Em `middlewares/`, ficam, bem... os [middlewares](https://expressjs.com/pt-br/guide/using-middleware.html). Se sua aplicação tem uma lógica que se repete por vários endpoints, considere mover essa responsabilidade para um middleware. Não se repita!
- Em `models/`, ficam os modelos de dados da aplicação, isto é - o formato de cada entidade, contendo seus atributos e respectivos tipos, além de funções de validação.
- Em `routes/`, ficam os arquivos responsáveis por expor as rotas da aplicação - eles ligam cada endpoint a seus respectivos métodos em algum controller, além de definir quais middlewares serão utilizados. 
- Em `utils/`, ficam funções utilitárias, que fogem ao escopo dos demais diretórios. Esses métodos costumam ser acessados por diferentes arquivos.



### Arquivos de inicialização
- `database.js` cria uma interface de comunicação com o banco de dados, definindo métodos para abrir e fechar a conexão.
- `router.js` cria um Router contendo todos os endpoints  expostos pelos arquivos em `routes/`.
- `app.js` cria o servidor, injetando nele os middlewares, endpoints, e um objeto de banco de dados.
- `index.js` é o ponto de entrada da aplicação. Ele chama as rotinas de inicialização, pondo o servidor de pé e iniciando a conexão com o banco de dados. Ele também é responsável por garantir que a aplicação [feche graciosamente](https://expressjs.com/en/advanced/healthcheck-graceful-shutdown.html).

## Criando novos módulos

Suponha que seu gerente acabou de lhe alocar em um novo projeto, e para sua primeira sprint lhe foi assinalada a tarefa de adicionar um módulo de produtos à aplicação.

O primeiro passo é criar o model, então, dentro de `models/`, crie um arquivo `Product.js`:

```js
import { Model, DataTypes } from 'sequelize';
import Joi from 'joi';

class Product extends Model {
    static init(sequelize) {
        super.init({
            name: {
                type: DataTypes.STRING,
                allowNull: false
            },
            price: {
                type: DataTypes.FLOAT,
                allowNull: false
            }
        });
    }
}

Product.rules = Joi.object({
	name: Joi.string().required(),
	price: Joi.number().min(0).required()
});

export default Product;
```

Definimos o _model_ `Product` e também usamos o Joi para criar um objeto de validação chamado `rules`, que injetamos na classe antes de exportá-la. Logo veremos como ele será utilizado. Agora, vamos criar  o `ProductController` em `controllers/`. Note que uma conexão ao banco de dados chamada `sequelize` é injetada através do método `init()` - essa injeção ocorre durante a inicialização da aplicação, no arquivo `database.js`.

```js
async getAll(req, res) {
	try {
	    const products = await this.Product.findAll();
	    return res.status(200).json(products);
	} catch ({ message }) {
	    return res.status(500).json({ message });
	}
}

async create(req, res) {
	try {
		const newProduct = await this.Product.create(req.body);
		return res.status(201).json(newProduct);
	} catch ({ message }) {
		return res.status(500).json({ message });
	}
}

export default { getAll, create };
```

Definimos as funções `getAll` e `create` e exportamos elas num objeto. Observe que no `create` assumimos que os dados enviados pelo cliente via `req.body` estão corretos, pois a validação já terá sido feita por um middleware, como veremos a seguir. 

O último passo é expor um endpoint para os métodos que criamos. Em `routes/`, crie um arquivo `productRoute.js`:

```js
import { Router } from 'express';
import ProductController from '../controllers/ProductController';
import Product from '../models/Product';
import validate from '../midddlewares/validate';

const router = Router();

router.get('/', ProductController.getAll);
router.post('/', validate(Product.rules), ProductController.create);

export default { name: 'product', router };
```

Criamos um _router_ que irá servir o endpoint `/product`. Então, associamos cada verbo HTTP a um método do controller, utilizando o middleware de validação quando necessário. Por fim, exportamos um objeto com o nome do endpoint que será exposto, e o router. Pronto! Não é preciso fazer mais nada, o `createRouter`  já se responsabilizará por incluir esses endpoints em `api/product`. 

## Dependências

### Produção

- [bcrypt](https://www.npmjs.com/package/bcrypt)
- [carrier](https://www.npmjs.com/package/carrier)
- [compression](https://www.npmjs.com/package/compression)
- [cors](https://www.npmjs.com/package/cors)
- [cross-env](https://www.npmjs.com/package/cross-env)
- [dotenv](https://www.npmjs.com/package/dotenv)
- [express](https://www.npmjs.com/package/express)
- [express-rate-limit](https://www.npmjs.com/package/express-rate-limit)
- [helmet](https://www.npmjs.com/package/helmet)
- [joi](https://www.npmjs.com/package/joi)
- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)
- [mongoose](https://www.npmjs.com/package/mongoose)
- [morgan](https://www.npmjs.com/package/morgan)
- [sequelize](https://www.npmjs.com/package/sequelize)
- [sqlite3](https://www.npmjs.com/package/sqlite3)

### Desenvolvimento

- [babel](https://www.npmjs.com/package/babel)
- [chai](https://www.npmjs.com/package/chai)
- [eslint](https://www.npmjs.com/package/eslint)
- [husky](https://www.npmjs.com/package/husky)
- [lint-staged](https://www.npmjs.com/package/lint-staged)
- [mocha](https://www.npmjs.com/package/mocha)
- [rewire](https://www.npmjs.com/package/rewire)
- [sinon](https://www.npmjs.com/package/sinon)
