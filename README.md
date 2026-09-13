# Predição de Reclamações

Aplicação web desenvolvida com [Streamlit](https://streamlit.io/) para estimar a probabilidade de uma unidade consumidora registrar uma reclamação nos próximos 15 dias.

O projeto utiliza uma rede neural MLP treinada com TensorFlow/Keras. A aplicação recebe características de consumo e de relacionamento do cliente, aplica os mesmos transformadores usados no treinamento e exibe uma classificação de risco.

## Visão geral

- **Tipo de problema:** classificação binária.
- **Classe 0:** não há previsão de reclamação nos próximos 15 dias.
- **Classe 1:** há previsão de reclamação nos próximos 15 dias.
- **Threshold operacional:** carregado de `config_modelo.pkl` (atualmente apresentado pela interface como `0,40`).
- **Interface:** Streamlit.
- **Modelo:** TensorFlow/Keras, salvo em `modelo_reclamacao_15d.keras`.
- **Sistema documentado neste guia:** Windows com PowerShell ou Prompt de Comando.

## Estrutura principal

```text
.
├── app.py                       # Interface Streamlit e fluxo de inferência
├── requirements.txt             # Dependências Python fixadas por versão
├── modelo_reclamacao_15d.keras  # Modelo neural treinado
├── scaler_num.pkl               # Normalizador das variáveis numéricas
├── encoder_classe.pkl           # Codificador da classe de consumo
├── encoder_municipio.pkl        # Codificador dos municípios
├── config_modelo.pkl            # Configurações da inferência, incluindo o threshold
├── base/                        # Bases e artefatos auxiliares do desenvolvimento


O arquivo `app.py` carrega o modelo e os artefatos diretamente da raiz do projeto. Por isso, os comandos devem ser executados a partir da pasta que contém `app.py`.

## Pré-requisitos

1. Windows 10 ou superior.
2. Python **3.13** instalado e disponível pelo comando `py`.
3. Git, caso o projeto ainda precise ser clonado.
4. Espaço em disco suficiente para instalar TensorFlow e as demais dependências.

Para conferir a versão do Python:

```powershell
py --version
```

Se o comando `py` não existir, instale o Python pelo site oficial: <https://www.python.org/downloads/>.

## Instalação passo a passo

### 1. Obter o projeto

Se o projeto ainda não estiver no computador:

```powershell
git clone https://github.com/Rafael-Soares/Previsao_reclamacoes.git
cd Previsao_reclamacoes
```

Se a pasta já estiver disponível, abra o PowerShell nela. Por exemplo:

```powershell
cd "C:\caminho\para\Previsao_reclamacoes"
```

Confirme que está na raiz correta:

```powershell
Test-Path .\app.py
Test-Path .\requirements.txt
```

Os dois comandos devem retornar `True`.

### 2. Criar a pasta virtual

O nome utilizado pelo projeto é `ambiente_virtual`:

```powershell
py -3.13 -m venv ambiente_virtual
```

Essa pasta está no `.gitignore` e não deve ser versionada.

### 3. Ativar o ambiente virtual

No PowerShell:

```powershell
.\ambiente_virtual\Scripts\Activate.ps1
```

Quando a ativação funcionar, o nome do ambiente aparecerá no início do prompt, por exemplo:

```text
(ambiente_virtual) PS C:\caminho\para\Previsao_reclamacoes>
```

Se o PowerShell bloquear a execução do script, permita a alteração somente para a sessão atual:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\ambiente_virtual\Scripts\Activate.ps1
```

Alternativamente, no Prompt de Comando (CMD):

```bat
ambiente_virtual\Scripts\activate.bat
```

### 4. Atualizar o instalador e instalar as dependências

Com o ambiente virtual ativado:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

O `requirements.txt` fixa as versões de Streamlit, TensorFlow, NumPy, scikit-learn e bibliotecas auxiliares para tornar a instalação reproduzível.

### 5. Executar a aplicação

Ainda na raiz do projeto e com o ambiente virtual ativado:

```powershell
python -m streamlit run app.py
```

O Streamlit exibirá um endereço local, normalmente:

```text
Local URL: http://localhost:8501
```

Abra esse endereço no navegador. Para encerrar a aplicação, volte ao terminal e pressione `Ctrl+C`.

## Como usar

1. Escolha a **classe de consumo**.
2. Escolha o **município**.
3. Informe o consumo atual (`M0`) e os consumos anteriores (`M1` e `M2`).
4. Informe a quantidade de reclamações nos últimos 30, 90 e 180 dias.
5. Informe a quantidade de serviços nos últimos 30 e 90 dias.
6. Clique em **Calcular risco**.
7. Consulte a probabilidade e a indicação de cliente de alto ou baixo risco.

Durante a inferência, o aplicativo calcula variáveis derivadas de consumo e flags de histórico antes de enviar os dados ao modelo. Os campos de mês/ano e algumas flags são preenchidos internamente pela aplicação.

## Comandos úteis

Verificar se o ambiente ativo é o esperado:

```powershell
python --version
python -c "import sys; print(sys.executable)"
```

Listar as dependências instaladas:

```powershell
python -m pip list
```

Desativar o ambiente virtual:

```powershell
deactivate
```

## Solução de problemas

### `ModuleNotFoundError`

Verifique se o ambiente está ativado. O caminho exibido por este comando deve conter `ambiente_virtual`:

```powershell
python -c "import sys; print(sys.executable)"
```

Se necessário, reinstale as dependências:

```powershell
python -m pip install -r requirements.txt
```

### `FileNotFoundError` para o modelo ou arquivos `.pkl`

Execute o Streamlit a partir da raiz do projeto, onde estão `app.py`, `modelo_reclamacao_15d.keras`, `scaler_num.pkl`, `encoder_classe.pkl`, `encoder_municipio.pkl` e `config_modelo.pkl`:

```powershell
cd "C:\caminho\para\Previsao_reclamacoes"
python -m streamlit run app.py
```

Não mova ou renomeie esses artefatos sem atualizar o código e validar a compatibilidade com o modelo treinado.

### TensorFlow exibe mensagens no terminal

Mensagens sobre CPU, oneDNN ou inicialização do TensorFlow podem aparecer no primeiro carregamento. Elas não indicam necessariamente uma falha. Verifique se o navegador abriu a aplicação e se o modelo foi carregado antes de investigar os logs.

### Porta 8501 já está em uso

Execute em outra porta:

```powershell
python -m streamlit run app.py --server.port 8502
```

## Observações para manutenção

- O modelo e os codificadores precisam permanecer compatíveis com a ordem e o formato das variáveis usados no treinamento.
- O código usa caminhos relativos; manter a execução na raiz do projeto evita erros de carregamento.
- O cálculo de `media_6m` é uma aproximação baseada nos três meses informados, conforme indicado nos comentários de `app.py`.
- Os arquivos dentro de `base/` são artefatos de dados e otimização do desenvolvimento; eles não são carregados diretamente pela interface atual.
- O repositório não possui uma suíte de testes automatizados configurada. Antes de alterar o pipeline de inferência, valide manualmente o carregamento do modelo e uma predição pela interface.

