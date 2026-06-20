# Agente IA — Imersão Alura

> Esse é meio que um tutorial documentado que fiz durante a Imersão de Agentes de IA da Alura, cobrindo as 3 aulas: criação do agente com RAG, integração com banco de dados MySQL, e conexão com Telegram. Inclui anotações sobre etapas que a aula não explicou (ou que meu cerebro não entendeu) e pontos de atenção identificados ao longo do processo. Usei bastante o auxilio do Claude code para tirar dúvidas sobre as aulas e de como documentar.
>
> **Meus primeiros documentos eram uma bagunça** 


---

## Sobre este guia

Esse material documenta o passo a passo seguido durante a imersão, usando as ferramentas oficiais do curso (Cohere, Railway, N8N Cloud, Telegram). Como esses serviços são pagos/com teste gratuito limitado, este guia irá servir como base para uma futura versão 100% local e gratuita que estou trabalhando para fins de aprendizado (LM Studio + N8N self-hosted + MySQL local).

**Stack original (Alura):** 

| Camada | Ferramenta |
|---|---|
| Orquestração | N8N (cloud) |
| LLM de chat | Cohere — Command-A-03-2025 |
| Embeddings | Cohere — Embed-Multilingual-v3.0 (1024 dimensões) |
| Banco de dados | MySQL via Railway |
| Interface de chat | Telegram |

![Visão geral do resultado final do agente completo](./assets/overview-final.gif)


---

## Aula 1 — Criando o Agente com RAG (Cohere + N8N)

### Pré-requisitos
Contas criadas em [cohere.com](https://cohere.com), [railway.com](https://railway.com), [n8n.io](https://n8n.io), e Telegram instalado (celular ou desktop).

### Gerando a API Key do Cohere
No site do Cohere, aba **API Keys** → **New Trial Key**.


<img width="1858" height="961" alt="image3" src="https://github.com/user-attachments/assets/124a607c-13d2-4a19-a015-5a592c95b964" />



> **Nota importante (não explicada pela Alura):** o vínculo da credencial as vezes não é automático. No N8N, ao adicionar qualquer nó Cohere ("Cohere Chat Model" ou "Embeddings Cohere"), ele pede para criar uma credencial nova — é nesse momento que se cola a API Key gerada acima.

### Etapa 1 — Carga de dados (Vector Store)

1. Criar workflow novo no N8N

   <img width="1913" height="990" alt="image2" src="https://github.com/user-attachments/assets/15b0c644-e9e9-4f8a-9dce-e30492f0df7d" />

2. Vá em add first step e selecione  **"Trigger Manually"** para criar o primeiro nó, pode fechar a janela.
3. Crie um segundo nó que se conectara ao primeiro, mas desta vez selecione **HTTP Request**, método **GET**
  <img width="364" height="313" alt="image" src="https://github.com/user-attachments/assets/271409c5-b308-47b4-990d-06ad4a0e18fd" /> <img width="482" height="225" alt="image" src="https://github.com/user-attachments/assets/30d7bf5c-b689-483c-8fac-fea8cbc65e7f" />


4. Na URL, cole o link **raw** do arquivo `.txt` fornecido pela Alura (ou o seu próprio, hospedado no GitHub).
     > **Link da alura**: https://raw.githubusercontent.com/ericmonne/chocolatech-imersao/refs/heads/main/Manual%20de%20RH%20ChocolaTech.txt
   
5. Antes de fechar vá em **"Add Option" → "Response"**, e selecione o formato **TXT** (isso ira evitar do sistema ler o arquivo do modo errado e dar erro)

<img width="906" height="884" alt="https txt" src="https://github.com/user-attachments/assets/c9d6b18f-d72f-4410-82b5-382f64e75523" />


> O que esse passo faz: a máquina acessa o link informado e traz o conteúdo do arquivo como resposta — é a "matéria-prima" que vai alimentar a base de conhecimento do agente.

6. Crie um novo nó vinculado ao https e selecione **Simple Vector Store** → modo **"Add Documents to Vector Store"**
7. Conecte os dois sub-nós obrigatórios:
   - **Embeddings** → Embeddings Cohere, Credencial → **cohere account**,  Model → **Embed-Multilingual-v3.0 (1024 Dimensions)**
   - **Document** → Default Data Loader (sem configuração extra)
    <img width="370" height="604" alt="cohere acc" src="https://github.com/user-attachments/assets/69965547-cea1-4ae3-a96e-ffd966404e2d" />



<img width="1718" height="581" alt="image1" src="https://github.com/user-attachments/assets/039447e3-ac03-48b3-8abe-7e5a5584e64b" />


> O modelo multilingual garante que o agente entenda textos em português, inglês ou qualquer outro idioma sem problemas de interpretação.

### Etapa 2 — Criando o chatbot

1. No n8n crie um novo workflow → primeiro nó: **"On Chat Message Received"** (sem configuração)
2. Conect o primeiro nó a outro selecionando **AI** → **AI Agent**, que abrirá 3 subs nós:
<img width="737" height="351" alt="image5" src="https://github.com/user-attachments/assets/f5830dbb-540a-4166-934d-df21ae91736a" />


**Chat Model** → Cohere, modelo **Command-a-03-2025**

**Memory** → Simple Memory (sem configuração)

**Tool** → Simple Vector Store (mesmo modelo do workflow anterior) e adicione uma descrição, irei usar o exemplo disponibilizado pela própria alura.
- Description:
  > "Busque informações sobre políticas de RH, férias, horários e rotinas da empresa Chocolatech"
- Embedding obrigatório → Cohere, modelo **Embed-Multilingual-v3.0 (1024 Dimensions)**

<img width="946" height="588" alt="image8" src="https://github.com/user-attachments/assets/01dcb669-14ce-4bc7-ac4b-090bc12308bb" />


---

## Aula 2 — Integração com Banco de Dados (MySQL via Railway)

> **Nota:** esta aula usa o Railway como hospedagem paga do MySQL. Em uma futura versão local, este banco pode ser substituído por um MySQL rodando localmente (via Docker ou instalação direta), eliminando o custo sem perder a funcionalidade.

### Criando o banco de dados no Railway

1. No Railway: **New → Database → MySQL** (a criação demora um pouco; mensagens de erro temporárias são normais)
<img width="412" height="480" alt="image10" src="https://github.com/user-attachments/assets/9e492da6-8575-49be-8007-6f50bbdd77e1" />
<img width="413" height="256" alt="image6" src="https://github.com/user-attachments/assets/9c0e6815-6b2a-487f-9e5f-9ebb86aaafa0" />





<img width="1179" height="292" alt="image17" src="https://github.com/user-attachments/assets/32c32c4b-f70f-467e-9040-4a0b3846c055" />

2. Quando finalizar, o console SQL irá aparecer, iremos usar o script fornecido pela alura e iremos pressionar **Enter**
<img width="1180" height="504" alt="image7" src="https://github.com/user-attachments/assets/773110d8-3138-4935-a991-e60a4b13fd1d" />


```sql
CREATE TABLE funcionarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE,
    departamento VARCHAR(100) NOT NULL,
    cargo VARCHAR(100) NOT NULL,
    data_admissao DATE NOT NULL,
    saldo_ferias INT NOT NULL DEFAULT 0,
    banco_horas DECIMAL(5,1) NOT NULL DEFAULT 0,
    regime VARCHAR(20) NOT NULL DEFAULT 'hibrido'
);
```

3. Após rodar o script você pode apagar tudo da caixa de seleção e iremos testar utilizando o comando `SELECT * FROM funcionarios;` (aqui estimamos um retorno vazio, pois ainda não há dados, se não der nenhum erro isso ira confirmar que a tabela foi criada)

<img width="201" height="33" alt="image4" src="https://github.com/user-attachments/assets/8eb5ae39-9e05-470c-9cf6-78a5383278bc" />

4. Agora na caixa de seleção, pode apagar tudo novamente e inserira o script abaixo:

```sql
INSERT INTO funcionarios (nome, email, departamento, cargo, data_admissao, saldo_ferias, banco_horas, regime) VALUES
('João Silva', 'joao.silva@empresa.com', 'Engenharia', 'Engenheiro de Software', '2022-03-10', 20, 0.0, 'hibrido'),
('Maria Souza', 'maria.souza@empresa.com', 'Recursos Humanos', 'Analista de RH', '2021-05-15', 5, 12.5, 'hibrido'),
('Carlos Oliveira', 'carlos.oliveira@empresa.com', 'Financeiro', 'Analista Financeiro', '2023-01-20', 0, 0.0, 'presencial'),
('Ana Lima', 'ana.lima@empresa.com', 'Marketing', 'Especialista em Marketing', '2020-11-05', 15, -4.0, 'remoto'),
('Pedro Santos', 'pedro.santos@empresa.com', 'Vendas', 'Executivo de Vendas', '2022-08-01', 10, 8.0, 'hibrido'),
('Fernanda Costa', 'fernanda.costa@empresa.com', 'Operações', 'Gerente de Operações', '2019-02-12', 30, 0.0, 'presencial'),
('Rafael Mendes', 'rafael.mendes@empresa.com', 'TI', 'Analista de Suporte', '2023-06-10', 0, 15.5, 'hibrido'),
('Juliana Rocha', 'juliana.rocha@empresa.com', 'Engenharia', 'Desenvolvedora Front-end', '2021-09-25', 12, 0.0, 'remoto'),
('Bruno Alves', 'bruno.alves@empresa.com', 'Design', 'Designer UX/UI', '2022-04-18', 8, 3.5, 'hibrido'),
('Camila Ferreira', 'camila.ferreira@empresa.com', 'Atendimento', 'Analista de Atendimento', '2024-01-05', 0, 0.0, 'hibrido'),
('Eric Monné', 'eric.monne@chocolatech.com', 'Produto', 'Instrutor de Cursos', '2024-01-15', 25, 8.0, 'hibrido');
```
> Isso irá inserir os dados dos funcionários no nosso sistema.

5. Podemos confirmar se está tudo certo com o mesmo comando `SELECT * FROM funcionarios;` que agora irá nos retornar os 11 funcionários cadastrados.

<img width="1166" height="851" alt="image9" src="https://github.com/user-attachments/assets/a6f42ace-413a-4034-af01-cb372136c91d" />


### Expondo o banco publicamente

1. Aba **Settings** → **Public Networking** → **Generate Domain**, mantendo a porta padrão `3306`
2. Copiar o domínio público gerado
<img width="556" height="95" alt="image5" src="https://github.com/user-attachments/assets/1b6798d1-cbbf-44a0-b0e1-b7c0e895b86a" />


> ⚠️ **Ponto de atenção de segurança:** este banco fica exposto publicamente com a senha padrão do Railway (`MYSQL_ROOT_PASSWORD`). Adequado apenas para fins de estudo de curta duração — não é uma configuração segura para produção ou uso prolongado.

### Conectando o N8N ao MySQL

1. Crie um nó adicional na tool e selecione **MySQL Tool** (vinculado ao AI Agent), clique em **"Set up credential"**
<img width="631" height="70" alt="image2" src="https://github.com/user-attachments/assets/12a6a114-68c4-4bb3-ae61-985f3fec8465" />

<img width="1202" height="742" alt="image8" src="https://github.com/user-attachments/assets/641a058e-4f81-4d82-8c22-aaa2ab806954" />

2. Vamos preencher essa tabela utilizando os dados:

   - **Host:** domínio público copiado do Railway, recorte os 4 numeros finais e apague o **":"**
   - **Port:** Cole os 4 dígitos finais do domínio 
   - **User:** `root`
   - **Password:** copiada da aba **Variables** do Railway, campo `MYSQL_ROOT_PASSWORD`
   <img width="1174" height="691" alt="image3" src="https://github.com/user-attachments/assets/9a1a0171-b11e-41e8-9f16-160aeae57ac1" />

   - **Database:** `railway`
   - Após isso é so clicar em salvar.


3. Agora dentro desta MYSQL tool iremos ir na aba **Operation** e selecionar Select. Na aba Table vá em funcionários, nota-se que irá abrir uma série de configurações.

<img width="908" height="886" alt="configurando 1" src="https://github.com/user-attachments/assets/3ab24310-bcab-4f82-8576-df947d10f846" />



4. Em **Select Rows** clique em **Add condition** e preencha as abas a seguir:
   - **Column:** `nome`
   - **Operator:** `LIKE` (permite correspondência aproximada — sem diferenciar maiúsculas, minúsculas ou acentuação)
   - **Value:** deixar a própria IA preencher dinamicamente
   <img width="649" height="129" alt="image11" src="https://github.com/user-attachments/assets/a48ad7fe-e94f-4d14-8572-e1eb3c573908" />

5. CLique em add description e insira esse exemplo fornecido da alura:
   > "Sempre use o formato %nome% (com % no início e no fim) para busca parcial. Exemplo: %João Silva%"

   <img width="620" height="314" alt="descrição do mysql" src="https://github.com/user-attachments/assets/8d734aff-d675-488a-8073-83ead58dfa11" />

Pode fechar, essa parte está finalizada.

> ⚠️ **Ponto de atenção de segurança:** o uso de `LIKE` com interpolação direta de texto, sem sanitização ou prepared statements, é a porta de entrada clássica para ataques de **SQL Injection**. Esta configuração é adequada apenas para fins didáticos.

### Refinando o comportamento do agente (System Message)

No nosso nó **AI Agent vamos em  Options → System Message** e adicionea mensagem:

> "Você é o HR Buddy, assistente virtual de RH da ChocolaTech.
>
> REGRAS:
> 1. Sempre responda em português.
> 2. Responda APENAS dúvidas relacionadas a RH.
>
> IDENTIFICAÇÃO DO FUNCIONÁRIO:
> - Se o usuário não disser quem é, pergunte o nome completo dele logo na primeira mensagem.
> - Use a ferramenta MySQL para buscar na tabela funcionarios usando SEMPRE o NOME COMPLETO informado pelo usuário na conversa.
> - Se encontrado: use os saldos de férias e banco de horas.
> - Se não encontrado: não invente dados pessoais. Responda apenas com base nas políticas gerais de RH do Vector Store.
>
> Use a base de conhecimento para dúvidas gerais."


---

## Aula 3 — Conectando nosso sistema ao Telegram

### Criando o bot no BotFather

1. Para criarmos nosso bot do telegram temos que acessar o link [web.telegram.org/k/#@BotFather](http://web.telegram.org/k/#@BotFather) (necessário ter conta criada previamente pelo app do celular)
2. Clicar em **Start** para começar uma conversa com o BotFather selecionar **New Bot**
<img width="785" height="970" alt="image5" src="https://github.com/user-attachments/assets/a878320e-2efe-4728-af3f-d6fcc8c815bc" />

3. Escolha um nome de exibição (ex da aula e que eu também usei: "HR Buddy")
4. Escolha um **username único**, terminado obrigatoriamente em `Bot`, não pode repetir nomes já usados por outros alunos nem o do exeemplo da aula, eu usei Zingaii_bot no meu caso.

5. Após isso irá gerar um **token** Copie ele, iremos usar para fazer a ponte do nosso sistema ao telegram.

### Preparando o workflow para o Telegram

> A aula recomenda duplicar o workflow original como backup antes de modificar — assim, qualquer erro não compromete a versão funcional anterior.

1. No workflow duplicado, remova o nó inical **"When chat message received"**, iremos substitui-lo
2. No espaço vazio do workflow, aperte **N** para abrir busca de nós → selecione **Telegram → On Message**
3. Em **"Set up credential"**, cole o seu token do bot gerado no BotFather
4. Pode salvar e fechar, seu workflow ficará assim:
<img width="1071" height="658" alt="image1" src="https://github.com/user-attachments/assets/285feaa3-4e8f-4f13-9a19-74539405b414" />


4. No seu navegador web cole o link do bot ( disponível na própria conversa com o BotFather ) e inicie uma a conversa, o `/start` é enviado automaticamente
<img width="412" height="243" alt="image10" src="https://github.com/user-attachments/assets/3bb00284-5537-405a-876f-ffc8c374fa7c" />


### Ajustando Memory e Agent para reconhecer o Telegram

**No nó do nosso workflow "Simple Memory":**
1. Troque o **Session ID** de "Connected Chat Trigger Node" para **"Define Below"**
2. No campo **Key**, selecione modo **Expression**
<img width="339" height="66" alt="image" src="https://github.com/user-attachments/assets/b6e07809-002f-4bea-b7f9-cd85a4afff32" />

3. Abra o console de expression, bem na direita você verá o incone do telegram, clique nele e arraste o campo **`chat → id`** até o campo da expression

<img width="1920" height="996" alt="test" src="https://github.com/user-attachments/assets/35133bc9-559e-4ed1-b29d-b2cd82ba3ab9" />




** nó do workflow  dentro do AI Agent:**
1. Usaremos a mesma lógica: trocar de "Connected Chat Trigger Node" para **"Define Below"**
2. Arrastar o campo **`text`** da mensagem do Telegram da direita para o console de expressão

<img width="920" height="858" alt="22222" src="https://github.com/user-attachments/assets/1106d855-746f-4e97-87b8-22bdd296e166" />



### Conectando a resposta de volta ao Telegram

1. No workflow vamos adicionar um novo nó Telegram à direita do AI Agent: **"Send Text Message"**
2. Arraste o **output do AI Agent** para o campo de texto da mensagem

<img width="1920" height="854" alt="text ai" src="https://github.com/user-attachments/assets/628deb4a-da4a-41e1-ace4-1cf484498dca" />


3. Arraste o **chat id** (do Telegram Trigger) para o campo correspondente de destino


<img width="1920" height="1080" alt="TRIGGER ID" src="https://github.com/user-attachments/assets/e981fcd6-cb7f-4f4a-ade2-39abf46d7752" />



> **Nota:** ao testar executando o workflow manualmente, ele entra em estado de carregamento infinito até que uma mensagem seja efetivamente enviada ao bot no Telegram — isso é esperado, não é erro.




https://github.com/user-attachments/assets/427add05-3635-4ede-be0d-e8e4c2cabddb




### Deixando o agente ativo permanentemente

Para que o agente funcione sem precisar clicar manualmente em "Execute Workflow" a cada teste, o workflow precisa ser publicado/ativado no N8N, tornando-o uma automação contínua e pública.

---

## Próximos passos que estou trabalhando para criar uma versão totalmente gratuita que roda localmente.

- Substituir o Cohere pelo LM Studio ( como os modelos Qwen2.5-7B + nomic-embed-text)
- Substituir Railway por MySQL local (Docker ou instalação direta), eliminando o custo de hospedagem
- Manter o Telegram como interface, já que o bot em si é gratuito e permanente
- Aplicar medidas de proteção de dados ( ainda não faço ideia de como fazer isso hushsuhsu)

  ### Resumo sobre o que eu entendi sobre tudo isso
  > Criamos um bot do telegram que responde baseado no nosso banco de dados ligados a um agente de IA com instruções de como agir, aonde pegar os dados, e como responder nesse pequeno experimento de "RH" da Chocolatec.
  Enfim é isso, espero que esse "pequeno" tuturial tenha te ajudado e que no mínimo você tenha entendido algo kkkk.
  
