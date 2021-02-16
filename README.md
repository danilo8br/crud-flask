![1_tJbEjxx-ne6uUzCbqueIWg](https://user-images.githubusercontent.com/51414398/107884088-91e73100-6ed1-11eb-98f7-fe244f9aed0b.png)

<h1 align="center"> Python + Flask </h1>

<p align="center"> Crud using SQLAlchemy flask with Python </p>

## :rocket: About the project 

I did this project to train my flask skills.

## :wrench: How it works?

The program will make requests of the type Get, Post, Put and Delete in Postman to the MySQL database usign the Python language.

## :thinking:  Why?

Just practice my flask skills.

## :warning: Prerequisites

- pip install flask
- pip install flask_sqlalchemy
- pip install mysql-connector-python
- pip install mysqlclient

## The code

### Libraries

```
from flask import Flask, Response, request
from flask_sqlalchemy import SQLAlchemy
import mysql.connector
import json
```

### Initializing the flask
```
app = Flask('__name__')
```

### Configuring the database

```
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = True
```

### Connecting to the MySQL database

```
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://seu_usuario:sua_senha@seu_host/seu_banco'
```

### Combining the application

```
db = SQLAlchemy(app)
```

### Creating the "Category" table

```
class Categoria(db.Model):
    id_categoria = db.Column(db.Integer, primary_key = True)
    tipo_categoria = db.Column(db.String(30))
    
    def to_json1(self):
        return {'tipo_categoria': self.tipo_categoria}
```

### Creating the "Products" table

```
class Produtos(db.Model):
    id = db.Column(db.Integer, primary_key = True)
    codigo = db.Column(db.String(50))
    preco = db.Column(db.String(50))
    descricao = db.Column(db.String(50))
    categoria_id = db.Column(db.Integer, db.ForeignKey('categoria.id_categoria'))
    # Relacionamento
    categoria = db.relationship('Categoria', foreign_keys = categoria_id)

    def to_json(self):
        return {'id': self.id, 'codigo': self.codigo, 'preco': self.preco, 'descricao': self.descricao}
```


### Creating the tables in the database

```
db.create_all()
```

### Register

```
@app.route('/produtos', methods=['POST'])
def cadastrar_produtos():
    body = request.get_json()
    try:
        cliente = Produtos(codigo=body['codigo'], preco=body['preco'], descricao=body['descricao'])
        cliente1 = Categoria(tipo_categoria=body['tipo_categoria'])
        db.session.add(cliente)
        db.session.add(cliente1)
        db.session.commit()
        return gera_response(201, 'cadastro', cliente.to_json(),'Cadastrado com sucesso')
    except Exception as e:
        print('Erro', e)
        return gera_response(400, "cliente", {}, "Erro ao cadastrar")
```

### Select

```
@app.route('/produtos', methods=['GET'])
def selecionar_produtos():
    produtos_objetos = Produtos.query.all()
    produtos_castegoria_objetos = Categoria.query.all()
    produtos_json = [produtos.to_json() for produtos in produtos_objetos]
    categoria_json = [categoria.to_json1() for categoria in produtos_castegoria_objetos]
    return gera_response(200, "produtos", produtos_json, categoria_json)

```

### Update

```
@app.route('/produtos/<id>/<id_categoria>', methods=['PUT'])
def atualiza_cadastro(id, id_categoria):
    produtos_objeto = Produtos.query.filter_by(id=id).first()
    categoria_objeto = Categoria.query.filter_by(id_categoria=id_categoria).first()
    body = request.get_json()

    try:
        if 'codigo' in body:
            produtos_objeto.codigo = body['codigo']
        if 'preco' in body:
            produtos_objeto.preco = body['preco']
        if 'descricao' in body:
            produtos_objeto.descricao = body['descricao']
        if 'tipo_categoria' in body:
            categoria_objeto.tipo_categoria = body['tipo_categoria']
        db.session.add(produtos_objeto)
        db.session.add(categoria_objeto)
        db.session.commit()
        return gera_response(200, 'produtos', produtos_objeto.to_json(), 
        categoria_objeto.to_json1(), 'Atualizado com sucesso')

    except Exception as e:
        print('Erro', e)
        return gera_response(400, 'produtos', {}, 'Erro ao atualizar')
```

### Delete

```
@app.route('/produtos/<id>/<id_categoria>', methods=['DELETE'])
def deleta_produto(id, id_categoria):
    produtos_objeto = Produtos.query.filter_by(id=id).first()
    categoria_objeto = Categoria.query.filter_by(id_categoria=id_categoria).first()
    try:
        db.session.delete(produtos_objeto)
        db.session.delete(categoria_objeto)
        db.session.commit()
        return gera_response(200, 'produtos', produtos_objeto.to_json(), 'Deletado com sucesso')
    except Exception as e:
        return gera_response(200, 'produtos', {}, 'Erro ao deletar')
```

### Return of json

```
def gera_response(status, nome_do_conteudo, conteudo, conteudo1,  mensagem=False):
    body = {}
    body[nome_do_conteudo] = conteudo
    if mensagem:
        body['mensagem'] = mensagem
    return Response(json.dumps(body), status=status, mimetype='application/json')
```

### Running the application

```
app.run()
```
