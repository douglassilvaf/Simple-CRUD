# Portfólio do Projeto - CRUD em python

## Sobre Mim
Olá, senhoras e senhores! Sou o Douglas, um desenvolvedor dedicado a criar soluções inovadoras. Este repositório apresenta um dos meus projetos recentes, cujo objetivo é carregar um CSV e inserir seus dados em um banco de dados SQLite, aplicando critérios de validação nos campos nome, email, telefone e endereço. Além disso, o sistema permite operações de CRUD (Create, Read, Update, Delete) para gerenciar contatos dentro deste ambiente. Abaixo, descrevo as tecnologias utilizadas e como cada uma pode ser executada. Sinta-se à vontade para explorar o código e entrar em contato para qualquer dúvida ou colaboração.

### Projeto: CRUD de Contatos em Banco de Dados SQLite

#### Descrição
Este projeto implementa um sistema CRUD (Create, Read, Update, Delete) para gerenciar contatos em um banco de dados SQLite. Ele permite inserir novos contatos, visualizar todos os contatos existentes, atualizar informações de um contato específico e excluir contatos do banco de dados. O código também inclui validações para os campos de nome, email, telefone e endereço durante a inserção e atualização dos contatos.

#### Tecnologias Utilizadas
- Python
- SQLite

#### Funcionalidades
- Inserir novos contatos no banco de dados SQLite
- Visualizar todos os contatos cadastrados
- Atualizar informações de contatos existentes
- Excluir contatos do banco de dados


#### Como Executar
1. Clone o repositório:
   ```bash
   git clone https://github.com/douglassilvaf/Simple-CRUD.git
   ```

2. Navegue até o diretório do projeto:
   ```bash
   cd Simple-CRUD
   ```

3. Execute o script Python:
   ```bash
   python CRUD.py
   ```

#### Código
```python
import csv
import sqlite3
import re

def validate_email(email):
    # Verifica se o email está no formato correto
    return re.match(r'^[^@]+@[^@]+\.[^@]+$', email)

def validate_phone(phone):
    # Verifica se o telefone contém apenas números, + e -
    return re.match(r'^[0-9+-]+$', phone)

def validate_address(address):
    # Verifica se o endereço tem pelo menos um caractere alfanumérico
    return re.match(r'[a-zA-Z0-9]+', address)

def process_csv(csv_filepath, db_filepath):
    connection = sqlite3.connect(db_filepath)
    cursor = connection.cursor()
    cursor.execute('''
                    CREATE TABLE IF NOT EXISTS pessoas (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        nome TEXT,
                        email TEXT,
                        telefone TEXT,
                        endereco TEXT
                    )
                ''')
    connection.commit()

    cursor.execute('SELECT * FROM pessoas')
    rows = cursor.fetchall()
    if rows:
        connection.close()
        return

    with open(csv_filepath, 'r') as file:
        reader = csv.DictReader(file)
        for row in reader:
            # Validação dos campos
            name = row['Name']
            email = row['Email']
            phone = row['Phone']
            address = row['Address']

            if ' ' not in name:
                print(f"Nome inválido: {name} (deve conter sobrenome)")
                continue

            if not validate_email(email):
                print(f"Email inválido: {email}")
                continue

            if not validate_phone(phone):
                print(f"Telefone inválido: {phone}")
                continue

            if not validate_address(address):
                print(f"Endereço inválido: {address}")
                continue

            # Insere os dados na tabela SQLite
            cursor.execute("INSERT INTO pessoas (nome, email, telefone, endereco) VALUES (?, ?, ?, ?)",
                           (name, email, phone, address))
    connection.commit()
    connection.close()

def inserir_contato(nome, email, telefone, endereco):
    conn = sqlite3.connect('bancoFinal.db')
    c = conn.cursor()
    c.execute("INSERT INTO pessoas (nome, email, telefone, endereco) VALUES (?, ?, ?, ?)",
              (nome, email, telefone, endereco))
    conn.commit()

    # Obter o ID do último registro inserido (lastrowid)
    last_id = c.lastrowid
    c.execute("SELECT * FROM pessoas WHERE id = ?", (last_id,))
    ultimo_contato = c.fetchone()

    conn.close()
    print(f"""Você inseriu o contato:
{ultimo_contato}""")
    return ultimo_contato 

def exibir_contatos():
    conn = sqlite3.connect('bancoFinal.db')
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM pessoas')
    rows = cursor.fetchall()
    for row in rows:
        print(row)
    conn.close()

def atualizar_contato(id_contato, contato):
    conn = sqlite3.connect('bancoFinal.db')
    preencher_update = ', '.join(f"{campo} = ?" for campo in contato.keys())
    sql = f"UPDATE pessoas SET {preencher_update} WHERE id = ?"
    cursor = conn.cursor()
    cursor.execute(sql, list(contato.values()) + [id_contato])
    conn.commit()
    conn.close()

def deletar_contato(id_contato):
    conn = sqlite3.connect('bancoFinal.db')
    cursor = conn.cursor()
    cursor.execute("DELETE FROM pessoas WHERE id = ?", (id_contato,))
    conn.commit()
    conn.close()

if __name__ == '__main__':
    csv_filepath = 'contacts_with_errors.csv'  # Substitua pelo caminho do seu arquivo CSV
    db_filepath = 'bancoFinal.db'  # Nome do arquivo SQLite
    process_csv(csv_filepath, db_filepath)

    while True:
        print("""
Olá, vamos gerenciar um banco de dados:
Para adicionar contatos digite 1:
Para visualizar contatos digite 2:
Para atualizar contatos digite 3:
Para deletar contatos digite 4:
Para sair do programa digite 5:
""")
        answer = int(input())

        if answer == 1:
            while True:
                nome_digitado = str(input("Digite o nome: "))
                if ' ' not in nome_digitado:
                    print(f"Nome inválido: {nome_digitado} (deve conter sobrenome)")
                else:
                    break
            while True:
                email_digitado = str(input("Digite o email: "))
                if not validate_email(email_digitado):
                    print(f"Email inválido: {email_digitado}")
                else:
                    break
            while True:
                telefone_digitado = str(input("Digite o telefone: "))


                if not validate_phone(telefone_digitado):
                    print(f"Telefone inválido: {telefone_digitado}")
                else:
                    break
            while True:
                endereco_digitado = str(input("Digite o endereço: "))
                if not validate_address(endereco_digitado):
                    print("Endereço inválido.")
                else:
                    break
            inserir_contato(nome_digitado, email_digitado, telefone_digitado, endereco_digitado)

        elif answer == 2:
            exibir_contatos()

        elif answer == 3:
            id_contato = int(input("Digite o ID do contato a ser atualizado: "))
            contato = {}
            if input("Atualizar nome? (s/n): ").lower() == 's':
                while True:
                    nome_digitado = str(input("Digite o nome: "))
                    if ' ' not in nome_digitado:
                        print(f"Nome inválido: {nome_digitado} (deve conter sobrenome)")
                    else:
                        contato['nome'] = nome_digitado
                        break
            if input("Atualizar email? (s/n): ").lower() == 's':
                while True:
                    email_digitado = str(input("Digite o email: "))
                    if not validate_email(email_digitado):
                        print(f"Email inválido: {email_digitado}")
                    else:
                        contato['email'] = email_digitado
                        break
            if input("Atualizar telefone? (s/n): ").lower() == 's':
                while True:
                    telefone_digitado = str(input("Digite o telefone: "))
                    if not validate_phone(telefone_digitado):
                        print(f"Telefone inválido: {telefone_digitado}")
                    else:
                        contato['telefone'] = telefone_digitado
                        break
            if input("Atualizar endereço? (s/n): ").lower() == 's':
                while True:
                    endereco_digitado = str(input("Digite o endereço: "))
                    if not validate_address(endereco_digitado):
                        print("Endereço inválido.")
                    else:
                        contato['endereco'] = endereco_digitado
                        break
            if contato:
                atualizar_contato(id_contato, contato)
            else:
                print("Nenhum campo selecionado para atualização.")

        elif answer == 4:
            id_contato = int(input("Digite o ID do contato a ser deletado: "))
            deletar_contato(id_contato)

        elif answer == 5:
            break
        else:
            print("Opção inválida. Tente novamente.")
```

Este é um sistema CRUD para gerenciar contatos em um banco de dados SQLite, com validações para os campos de entrada e operações para inserir, visualizar, atualizar e deletar contatos.
