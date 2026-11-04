# Maria José — Assistente Financeiro Virtual
## Projeto para GitHub + Vercel (sem Google Apps Script)

Este pacote é um site independente hospedado na **Vercel**, com API em Node.js e **uma única planilha Google Sheets** como banco de dados. A interface principal é uma calculadora com campos automáticos; salvar um produto é opcional.

### 1. Arquivos

- `public/index.html`: interface, calculadora, logo manuscrito e painel de produtos;
- `api/index.js`: servidor, login, criação de empresas e comunicação com Google Sheets;
- `package.json`: dependências;
- `vercel.json`: configuração da publicação;
- `.env.example`: nomes das variáveis de ambiente;
- `.gitignore`: evita enviar credenciais ao GitHub.

### 2. Preparar a planilha Google (uma vez)

1. Crie uma planilha em https://sheets.google.com. Nome sugerido: `MARIA JOSÉ — BANCO DE PRECIFICAÇÃO`.
2. Copie o **ID da planilha**, no endereço `https://docs.google.com/spreadsheets/d/ID_DA_PLANILHA/edit`.
3. Não é necessário criar abas manualmente. O aplicativo criará `_EMPRESAS` e, depois, uma aba `EMP_...` para cada novo cliente.
4. **Não compartilhe a planilha com os clientes.** Apenas a conta de serviço deve ter acesso de edição.

### 3. Criar a conta de serviço Google (uma vez)

1. Entre em https://console.cloud.google.com/ e crie um projeto (ou escolha um já existente).
2. Em **APIs e serviços > Biblioteca**, habilite **Google Sheets API**. O projeto Google Cloud e a conta de serviço podem estar sujeitos a políticas/quotas da sua conta.
3. Em **IAM e administrador > Contas de serviço**, crie uma conta de serviço (ex.: `maria-jose-app`).
4. Abra a conta criada, vá em **Chaves > Adicionar chave > Criar nova chave > JSON**. Guarde o arquivo JSON de forma privada. Nunca envie esse arquivo ao repositório ou a terceiros.
5. No JSON localize `client_email`. Compartilhe a planilha do passo 2 com esse e-mail, **como Editor**.
6. O projeto usa o conteúdo integral desse JSON apenas como **variável secreta do servidor** (`GOOGLE_SERVICE_ACCOUNT_JSON`), nunca no navegador.

> Atenção: algumas organizações impedem a geração de chaves JSON para contas de serviço. Nesse caso, será preciso configurar uma forma de autenticação de servidor alternativa antes da implantação.

### 4. Subir os arquivos ao GitHub

1. Crie um repositório **privado** novo em https://github.com/new, nome sugerido `maria-jose-precificacao`.
2. Envie para a raiz do repositório os arquivos e as pastas deste pacote, preservando a estrutura `api/` e `public/`.
3. **Não envie o JSON da conta de serviço**, códigos de acesso reais ou arquivos `.env`.

### 5. Publicar no Vercel

1. Acesse https://vercel.com/new e importe o repositório GitHub recém-criado.
2. Se solicitado, use **Other** como framework. Diretório raiz: a raiz deste projeto.
3. No painel **Settings > Environment Variables**, cadastre estas quatro variáveis (em `Production`; também em `Preview` se desejar testar prévias):

| Variável | Valor |
|---|---|
| `GOOGLE_SHEET_ID` | ID copiado da URL da planilha |
| `GOOGLE_SERVICE_ACCOUNT_JSON` | **Todo o conteúdo JSON** da chave da conta de serviço, em uma única variável; preserve os caracteres de quebra de linha codificados `\\n` no campo `private_key` |
| `ADMIN_CODE` | Um código administrativo longo e exclusivo, por exemplo, gerado por um gerenciador de senhas; **não use o exemplo do manual** |
| `SESSION_SECRET` | Outra sequência aleatória de pelo menos 32 caracteres, diferente do código administrativo |

4. Clique em **Deploy**. Se adicionou variáveis depois do deploy, execute **Redeploy** para aplicá-las.
5. Abra o endereço publicado (algo como `https://maria-jose-precificacao.vercel.app`).
6. Faça login usando o `ADMIN_CODE` que você cadastrou na Vercel.
7. O app criará a aba `_EMPRESAS` na primeira consulta administrativa. Cadastre uma empresa no painel **Empresas**. O sistema criará uma aba exclusiva e exibirá um código aleatório de acesso ao cliente. **Copie esse código imediatamente**, pois não haverá consulta posterior; é possível gerar outro código, invalidando o antigo para novos logins.
8. Use uma janela anônima e acesse com o código da empresa para testar a experiência do cliente.

### 6. Como o cliente trabalha

1. Abre o endereço do site e informa seu código de acesso **sem lista de empresas**.
2. A calculadora aparece diretamente. Preenche custo direto em reais e despesas administrativas, financeiras, tributárias e comerciais como percentuais sobre a venda.
3. Sempre que começar a preencher a última linha de uma categoria, uma nova linha vazia surge automaticamente.
4. O preço sugerido, lucro e markup são atualizados imediatamente. O preço praticado pode ser simulado.
5. Pode clicar em **Salvar produto** ou simplesmente usar a calculadora sem cadastrar.
6. Em **Produtos salvos**, pode pesquisar, editar e excluir os produtos da própria empresa.

### 7. Fórmula aplicada

`PREÇO SUGERIDO = CUSTOS DIRETOS / [1 - (DESPESAS % + MARGEM DE LUCRO %) / 100]`

- **Custos diretos**: valores por unidade em reais;
- **Despesas e impostos**: percentuais sobre a receita de venda;
- **Lucro desejado**: margem sobre o preço de venda, e não um simples acréscimo sobre o custo;
- Se os percentuais somarem 100% ou mais, o preço não é calculável e o aplicativo alerta.
- As despesas fixas só devem ser lançadas em percentual após definição de um critério de rateio apropriado; esta versão não realiza rateio mensal automático.
- Antes de decisões financeiras reais, sua mãe deve validar o tratamento dos custos e tributos de cada cliente.

### 8. Segurança e cuidados

- Cada empresa tem **uma aba** em uma planilha privada; a autorização ocorre na API, não simplesmente escondendo abas no navegador.
- As credenciais do Google só existem no servidor, como variáveis privadas do Vercel.
- O código de acesso funciona como uma **senha**. Ele deve ser entregue individualmente ao cliente. Não use nomes fáceis de adivinhar.
- O código do cliente não aparece em listas. O administrador consegue gerar um novo código.
- As sessões expiram após 6 horas. Trocar o código bloqueia novos logins com o código antigo; sessões anteriores da empresa também passam a ser recusadas nas operações de leitura e gravação.
- A primeira versão **não implementa limitador persistente de tentativas de login**. Antes de disponibilizar publicamente, configure proteção de bots/rate limiting no Vercel ou um serviço equivalente; faça piloto apenas com usuários autorizados.
- Mantenha o repositório privado e o acesso ao projeto Vercel restrito.
- Faça cópias de segurança periódicas do Google Sheets, sobretudo antes de alterar a estrutura.
- Não há proteção transacional para edições simultâneas do mesmo produto. Evite que duas pessoas alterem o mesmo registro ao mesmo tempo.
- A Vercel e a Google Cloud possuem quotas e políticas de preço que podem variar; para poucos acessos, é possível começar em planos gratuitos, **sem garantia de gratuidade irrestrita**.

### 9. Solução de problemas

- **Erro de configuração**: confira os 4 nomes de variáveis do Vercel e faça redeploy.
- **Sem permissão na planilha**: confirme que compartilhou com o `client_email` correto do JSON como Editor.
- **Planilha não encontrada**: confira `GOOGLE_SHEET_ID` e se a Google Sheets API está ativada.
- **Erro ao fazer login**: use o código administrativo exato ou um código gerado pelo sistema; o nome da empresa não é a senha.
- **Mudanças no GitHub não aparecem**: espere concluir o novo deploy e abra o domínio de produção correto.

### 10. Arquitetura

`Cliente no navegador → Vercel (site) → Vercel Serverless API → Google Sheets API → planilha única (aba por empresa)`

**Não utiliza Google Apps Script.**
