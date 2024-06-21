PLT para PDF 
Descrição
Este projeto é uma automação em Python que transforma arquivos .plt gerados pelo Audaces em arquivos PDF. Seja para modelagem ou risco, essa ferramenta é ideal para confecções que precisam visualizar e imprimir moldes com precisão.

Funcionalidades
Leitura de comandos HPGL de arquivos .plt.
Ajuste de escala e centralização do desenho.
Geração de PDFs de alta resolução.
Conversão automática de todos os arquivos .plt em uma pasta.
Requisitos
Python 3.x
Bibliotecas os, re, matplotlib
Instalação
Clone o repositório:

bash
Copiar código
git clone https://github.com/seu-usuario/seu-repositorio.git
Navegue até o diretório do projeto:

bash
Copiar código
cd seu-repositorio
Instale a biblioteca matplotlib:

bash
Copiar código
pip install matplotlib
Uso
Defina as pastas de entrada e saída no script:

python
Copiar código
input_folder = r'C:\caminho\para\entrada'
output_folder = r'C:\caminho\para\saida'
Execute o script:

bash
Copiar código
python seu_script.py
Os PDFs gerados estarão na pasta de saída especificada.

Exemplo de Código
python
Copiar código
import os
import re
import matplotlib.pyplot as plt

def parse_hpgl(plt_file_path):
    commands = []
    with open(plt_file_path, 'r') as file:
        data = file.read().strip()
        parts = re.split(r'(?=[A-Za-z]{2})', data)
        for part in parts:
            if part:
                cmd = part[:2]
                args = re.findall(r'-?\d+', part[2:])
                commands.append((cmd, args))
    return commands

def plot_hpgl(commands):
    fig, ax = plt.subplots(figsize=(30, 30))
    ax.invert_yaxis()
    x, y = 0, 0
    
    pen_down = False
    min_x, min_y = float('inf'), float('inf')
    max_x, max_y = float('-inf'), float('-inf')

    for cmd, args in commands:
        if cmd in ['PU', 'PD', 'PA'] and args:
            for i in range(0, len(args), 2):
                px, py = int(args[i]), int(args[i+1])
                min_x, min_y = min(min_x, px), min(min_y, py)
                max_x, max_y = max(max_x, px), max(max_y, py)

    drawing_width = max_x - min_x
    drawing_height = max_y - min_y

    scale_x = 3000 / drawing_width
    scale_y = 3000 / drawing_height
    scale = min(scale_x, scale_y)

    offset_x = (3000 - drawing_width * scale) / 2
    offset_y = (3000 - drawing_height * scale) / 2

    for cmd, args in commands:
        if cmd == 'PU':
            pen_down = False
            if args:
                x, y = int(args[0]), int(args[1])
        elif cmd == 'PD':
            pen_down = True
            for i in range(0, len(args), 2):
                new_x, new_y = int(args[i]), int(args[i+1])
                if pen_down:
                    plt.plot([(x - min_x) * scale + offset_x, (new_x - min_x) * scale + offset_x],
                             [(y - min_y) * scale + offset_y, (new_y - min_y) * scale + offset_y], 'k-')
                x, y = new_x, new_y
        elif cmd == 'PA':
            if args:
                x, y = int(args[0]), int(args[1])

    plt.xlim(0, 3000)
    plt.ylim(0, 3000)
    plt.axis('off')

def convert_plt_to_pdf(plt_file_path, pdf_file_path):
    commands = parse_hpgl(plt_file_path)
    plot_hpgl(commands)
    plt.savefig(pdf_file_path, format='pdf')
    plt.close()

def convert_all_plt_in_folder(input_folder, output_folder):
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)
    
    for filename in os.listdir(input_folder):
        if filename.lower().endswith('.plt'):
            plt_file_path = os.path.join(input_folder, filename)
            pdf_file_path = os.path.join(output_folder, f"{os.path.splitext(filename)[0]}.pdf")
            convert_plt_to_pdf(plt_file_path, pdf_file_path)

input_folder = r'C:\Users\roque rafael\Desktop\projeto_py\entrada'
output_folder = r'C:\Users\roque rafael\Desktop\projeto_py\saida'

convert_all_plt_in_folder(input_folder, output_folder)
Contribuição
Contribuições são bem-vindas! Sinta-se à vontade para abrir issues e pull requests para sugerir melhorias ou correções.

Licença
Este projeto está licenciado sob a licença MIT. Consulte o arquivo LICENSE para obter mais informações.
