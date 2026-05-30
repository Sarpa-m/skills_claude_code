# **Skill: commit-and-docs-guidelines**

Este documento define as diretrizes estritas que o agente deve seguir ao escrever, refatorar, comentar e realizar commits de código. O objetivo principal é manter a base de código limpa, legível e profissional, utilizando as melhores práticas de engenharia de software.

## **1\. Diretrizes de Commits**

O agente deve gerar mensagens de commit claras, rastreáveis e baseadas no padrão **Conventional Commits**.

### **Estrutura do Commit**

\<tipo\>\[escopo opcional\]: \<descrição curta no imperativo\>

\[corpo opcional explicando o 'porquê' e não o 'como'\]

### **Tipos Permitidos**

* **feat:** Adição de uma nova funcionalidade.  
* **fix:** Correção de um bug.  
* **refactor:** Mudança de código que não corrige bug nem adiciona funcionalidade (ex: renomear variáveis, simplificar lógica).  
* **docs:** Alterações apenas na documentação.  
* **chore:** Atualização de dependências, configurações de build, etc.  
* **test:** Adição ou correção de testes.

### **Boas Práticas para Commits**

* Use o tempo verbal no imperativo na descrição (ex: "Adiciona validação de email" em vez de "Adicionado" ou "Adicionando").  
* Mantenha a primeira linha com no máximo 72 caracteres.  
* Se o commit for complexo, adicione um corpo explicando a motivação da mudança e os estados afetados.

## **2\. Diretrizes de Comentários no Código**

**Regra de Ouro:** O código deve ser o mais autoexplicativo possível através de bons nomes. Comentários devem explicar o **"porquê"** de uma decisão técnica, e não o **"o quê"** o código está fazendo.

* **Seja conciso e objetivo:** Não exagere nos comentários. Evite redundâncias.  
* **Estruturação:** Use padrões reconhecidos da linguagem (ex: JSDoc para JavaScript/TypeScript, Docstrings para Python).  
* **Idioma:** Todos os comentários devem ser escritos em **Português**, a menos que o projeto exija explicitamente outro idioma.

## **3\. Documentando Funções, Variáveis e Estados**

### **3.1. Variáveis**

* **Nomenclatura:** Use nomes descritivos e pronunciáveis que revelem a intenção (tempoExpiracaoSessao em vez de tEs).  
* **Comentários em variáveis:** Comente apenas se a variável armazenar um valor mágico, uma regra de negócio específica ou uma conversão complexa.  
* *Exemplo:*  
  // Fator de conversão de milissegundos para dias (Regra de faturamento)  
  const FATOR\_DIAS\_FATURAMENTO \= 86400000;   
  let usuarioAtivo \= true; // Não comentar: o nome já é autoexplicativo

### **3.2. Funções**

* Toda função pública ou complexa deve ter um bloco de comentário estruturado.  
* Descreva brevemente o propósito, os parâmetros esperados e o retorno.  
* *Exemplo:*  
  /\*\*  
   \* Calcula o desconto aplicável com base no histórico de compras do cliente.  
   \* \* @param {number} totalCompra \- O valor total da compra atual.  
   \* @param {number} pontosFidelidade \- Saldo atual de pontos do cliente.  
   \* @returns {number} O valor final com o desconto aplicado.  
   \*/  
  function calcularDescontoFidelidade(totalCompra: number, pontosFidelidade: number): number {  
      // Lógica de cálculo...  
  }

### **3.3. Estados do Código (State Management)**

* Quando lidando com gerenciamento de estado (ex: React state, Redux, ou máquinas de estado), documente claramente as transições esperadas e os valores iniciais.  
* Se um estado derivar de cálculos complexos, explique a dependência.  
* *Exemplo:*  
  /\*\*  
   \* Estado da requisição de pagamento.  
   \* Transições válidas: 'idle' \-\> 'loading' \-\> 'success' | 'error'  
   \*/  
  const \[statusPagamento, setStatusPagamento\] \= useState\<'idle' | 'loading' | 'success' | 'error'\>('idle');

  // O token só é gerado se o usuário estiver autenticado e o status for 'success'  
  const tokenSessao \= useMemo(() \=\> gerarToken(usuario, statusPagamento), \[usuario, statusPagamento\]);

## **4\. Checklist de Autoavaliação do Agente**

Antes de finalizar a resposta ou o commit, o agente deve validar:

* \[ \] O código resolve o problema proposto de forma eficiente?  
* \[ \] As variáveis e funções possuem nomes claros?  
* \[ \] Os comentários são estritamente necessários, concisos e explicam regras ou motivos não óbvios?  
* \[ \] A mensagem de commit segue o padrão Conventional Commits?  
* \[ \] O idioma utilizado nos comentários e explicações está correto (Português)?
