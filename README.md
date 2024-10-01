# trabalho-de-extens-o-python
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt
import random
import time


# Função para criar tabelas no banco de dados SQLite
def criar_tabelas():
    conn = sqlite3.connect('estoque.db')
    cursor = conn.cursor()

    # Tabela de produtos
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS produtos (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nome TEXT,
        quantidade INTEGER
    )
    ''')

    # Tabela de movimentação (entradas e saídas de estoque)
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS movimentacoes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        produto_id INTEGER,
        tipo TEXT,  -- 'entrada' ou 'saida'
        quantidade INTEGER,
        data TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        FOREIGN KEY (produto_id) REFERENCES produtos(id)
    )
    ''')

    conn.commit()
    conn.close()


# Função para adicionar um novo produto
def adicionar_produto(nome, quantidade):
    conn = sqlite3.connect('estoque.db')
    cursor = conn.cursor()
    cursor.execute("INSERT INTO produtos (nome, quantidade) VALUES (?, ?)", (nome, quantidade))
    conn.commit()
    conn.close()


# Função para registrar uma movimentação (entrada ou saída de estoque)
def registrar_movimentacao(produto_id, tipo, quantidade):
    conn = sqlite3.connect('estoque.db')
    cursor = conn.cursor()

    # Atualizar quantidade no estoque
    if tipo == 'entrada':
        cursor.execute("UPDATE produtos SET quantidade = quantidade + ? WHERE id = ?", (quantidade, produto_id))
    elif tipo == 'saida':
        cursor.execute("UPDATE produtos SET quantidade = quantidade - ? WHERE id = ?", (quantidade, produto_id))

    # Registrar movimentação
    cursor.execute("INSERT INTO movimentacoes (produto_id, tipo, quantidade) VALUES (?, ?, ?)",
                   (produto_id, tipo, quantidade))

    conn.commit()
    conn.close()


# Função para consultar o estoque atual
def consultar_estoque():
    conn = sqlite3.connect('estoque.db')
    df = pd.read_sql_query("SELECT * FROM produtos", conn)
    conn.close()
    return df


# Função para consultar movimentações e gerar gráfico
def consultar_movimentacoes(produto_id):
    conn = sqlite3.connect('estoque.db')
    df = pd.read_sql_query("SELECT * FROM movimentacoes WHERE produto_id = ?", conn, params=(produto_id,))
    conn.close()

    df['data'] = pd.to_datetime(df['data'])
    df['quantidade_acumulada'] = df.apply(
        lambda row: row['quantidade'] if row['tipo'] == 'entrada' else -row['quantidade'], axis=1).cumsum()

    plt.figure(figsize=(10, 5))
    plt.plot(df['data'], df['quantidade_acumulada'], marker='o')
    plt.title(f'Movimentação de Estoque do Produto {produto_id}')
    plt.xlabel('Data')
    plt.ylabel('Quantidade Acumulada')
    plt.grid(True)
    plt.show()


# Função para simular movimentações automáticas
def simular_movimentacao_automatica(produto_id, n_movimentacoes):
    for _ in range(n_movimentacoes):
        tipo = random.choice(['entrada', 'saida'])
        quantidade = random.randint(1, 50)
        registrar_movimentacao(produto_id, tipo, quantidade)
        print(f"Movimentação registrada: {tipo} de {quantidade} unidades para o produto {produto_id}")
        time.sleep(1)


# Função principal
def controle_de_estoque():
    criar_tabelas()

    # Adicionar um produto
    adicionar_produto("Parafusos", 100)
    adicionar_produto("Òleo", 50)
    adicionar_produto("Manete", 25)
    adicionar_produto("Paralama", 2)


    # Simular movimentações automáticas
    produto_id = 1
    produto_id = 2
    produto_id = 3
    produto_id = 4
    # Supondo que o produto 'Parafusos' tem o ID 1
    simular_movimentacao_automatica(produto_id, 10)

    # Consultar e mostrar o estoque atual
    estoque_df = consultar_estoque()
    print("Estoque Atual:")
    print(estoque_df)

    # Consultar e mostrar gráfico das movimentações
    consultar_movimentacoes(produto_id)


if _name_ == "_main_":
    controle_de_estoque()
