# IA na Aprendizagem de Novas Linguas

## 📒 Descrição
Essa IA usa o conhecimento do GPT-4 para te ensinar algumas palavras e como utilizar ela em algumas frases em idiomas novos, podendo ser utilizado diretamente do seu idioma nativo

## 🤖 Tecnologias Utilizadas
GPT-4 e python

## 🧐 Processo de Criação
Eu sempre gostei de aprender idiomas, então queria uma maneira mais simples de aprender pelo menos frases basicas para me comunicar com pessoas ao redor do mundo, sendo assim pensei nesse modelo, utilizei um pouco do meu conhecimento de python e uma ajuda das ias pra aperfeiçoar o código, essa é apenas uma versão basica, pretendo melhorar, adicionar mais idiomas.

## 🚀 Resultados
Não tenho resultados reais, pois apenas eu utilizei, achei que me ajudou muito na apredizagem de palavras basicas e comunicação minima

## Código

import tkinter as tk
from tkinter import ttk, messagebox
from googletrans import Translator
import sqlite3

# Inicializa o tradutor
translator = Translator()

# Função para criar o banco de dados e as tabelas
def criar_banco():
    conexao = sqlite3.connect("tradutor.db")
    cursor = conexao.cursor()
    
    # Criação da tabela de palavras
    cursor.execute('''CREATE TABLE IF NOT EXISTS Palavras (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        palavra TEXT NOT NULL,
                        idioma TEXT NOT NULL
                    )''')
    
    # Criação da tabela de frases de exemplo
    cursor.execute('''CREATE TABLE IF NOT EXISTS Frases (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        palavra_id INTEGER,
                        frase TEXT NOT NULL,
                        FOREIGN KEY(palavra_id) REFERENCES Palavras(id)
                    )''')
    
    conexao.commit()
    conexao.close()

# Função para adicionar uma palavra e suas frases de exemplo ao banco de dados
def adicionar_palavra(palavra, idioma, frases):
    conexao = sqlite3.connect("tradutor.db")
    cursor = conexao.cursor()
    
    # Insere a palavra e obtém o ID gerado
    cursor.execute("INSERT INTO Palavras (palavra, idioma) VALUES (?, ?)", (palavra, idioma))
    palavra_id = cursor.lastrowid
    
    # Insere as frases associadas a essa palavra
    for frase in frases:
        cursor.execute("INSERT INTO Frases (palavra_id, frase) VALUES (?, ?)", (palavra_id, frase))
    
    conexao.commit()
    conexao.close()

# Função para buscar frases de exemplo de uma palavra no banco de dados
def buscar_frases(palavra, idioma):
    conexao = sqlite3.connect("tradutor.db")
    cursor = conexao.cursor()
    
    # Busca o ID da palavra no idioma específico
    cursor.execute("SELECT id FROM Palavras WHERE palavra = ? AND idioma = ?", (palavra, idioma))
    resultado_palavra = cursor.fetchone()
    
    if not resultado_palavra:
        return []
    
    palavra_id = resultado_palavra[0]
    
    # Busca as frases associadas ao ID da palavra
    cursor.execute("SELECT frase FROM Frases WHERE palavra_id = ?", (palavra_id,))
    frases = [linha[0] for linha in cursor.fetchall()]
    
    conexao.close()
    return frases

# Função de tradução e exibição de frases de exemplo
def traduzir_palavra():
    idioma_nativo = idiomas[idioma_nativo_var.get()]
    idioma_aprendizado = idiomas[idioma_aprendizado_var.get()]
    palavra = entrada_palavra.get().strip()
    
    if not palavra:
        messagebox.showwarning("Aviso", "Digite uma palavra para traduzir.")
        return
    
    try:
        # Realiza a tradução da palavra
        traducao = translator.translate(palavra, src=idioma_nativo, dest=idioma_aprendizado)
        label_traducao.config(text=f"Tradução: {traducao.text}")
        
        # Exibe frases de exemplo para a palavra
        area_exemplo.delete(1.0, tk.END)
        exemplos = buscar_frases(palavra.lower(), idioma_nativo)
        
        if exemplos:
            for frase in exemplos:
                frase_traduzida = translator.translate(frase, src=idioma_nativo, dest=idioma_aprendizado)
                area_exemplo.insert(tk.END, f"- {frase_traduzida.text}\n")
        else:
            area_exemplo.insert(tk.END, "Nenhum exemplo disponível para esta palavra.")
            
    except Exception as e:
        messagebox.showerror("Erro", f"Erro ao traduzir: {e}")

# Inicializa o banco de dados
criar_banco()

# Configuração da interface gráfica
janela = tk.Tk()
janela.title("Tradutor com Frases de Exemplo")
janela.geometry("400x350")

# Dicionário de idiomas disponíveis
idiomas = {
    'Português': 'pt',
    'Inglês': 'en',
    'Espanhol': 'es',
    'Francês': 'fr',
}

# Widgets de seleção de idiomas
idioma_nativo_var = tk.StringVar(value='Português')
idioma_aprendizado_var = tk.StringVar(value='Inglês')

ttk.Label(janela, text="Idioma Nativo:").grid(row=0, column=0, padx=10, pady=10)
idioma_nativo_menu = ttk.Combobox(janela, textvariable=idioma_nativo_var, values=list(idiomas.keys()))
idioma_nativo_menu.grid(row=0, column=1, padx=10, pady=10)

ttk.Label(janela, text="Idioma de Aprendizado:").grid(row=1, column=0, padx=10, pady=10)
idioma_aprendizado_menu = ttk.Combobox(janela, textvariable=idioma_aprendizado_var, values=list(idiomas.keys()))
idioma_aprendizado_menu.grid(row=1, column=1, padx=10, pady=10)

# Entrada de palavra
ttk.Label(janela, text="Palavra:").grid(row=2, column=0, padx=10, pady=10)
entrada_palavra = ttk.Entry(janela, width=20)
entrada_palavra.grid(row=2, column=1, padx=10, pady=10)

# Botão de tradução
botao_traduzir = ttk.Button(janela, text="Traduzir", command=traduzir_palavra)
botao_traduzir.grid(row=3, column=0, columnspan=2, pady=10)

# Área para mostrar a tradução e exemplos
label_traducao = ttk.Label(janela, text="Tradução: ")
label_traducao.grid(row=4, column=0, columnspan=2, padx=10, pady=10)

ttk.Label(janela, text="Exemplos de Uso:").grid(row=5, column=0, columnspan=2, padx=10, pady=10)
area_exemplo = tk.Text(janela, width=40, height=5)
area_exemplo.grid(row=6, column=0, columnspan=2, padx=10, pady=10)

# Executa a interface
janela.mainloop()


