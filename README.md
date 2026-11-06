<div align="center">

# Instituto Federal do Piauí – IFPI Campus Piripiri
### Curso Superior de Tecnologia em Análise e Desenvolvimento de Sistemas (TADS)
### Disciplina: Tópicos Especiais em Programação
### Docente: Prof. Me. Maykol Sampaio
### Discente: Héber Bringel

---

## 📰 Pipeline Autônomo de Notícias Tech por E-mail (n8n & RSS)
**Unidade III: Orquestração de Automações & Arquitetura Orientada a Eventos (EDA)**

</div>

---

## 📌 Contextualização e Problema

No cenário corporativo moderno, o acompanhamento manual de atualizações em blogs, portais e documentações técnicas consome tempo e gera retrabalho. 

Esta atividade prática tem como objetivo desenvolver um **pipeline autônomo e agendado no n8n** para monitorar fontes de notícias públicas via protocolo RSS, filtrar as 3 matérias mais recentes e disparar um **relatório consolidado por e-mail** contendo o título e o link direto de cada uma das 3 notícias em uma única mensagem diária.

---

## 🔄 Arquitetura do Workflow (4 Etapas)

O pipeline foi projetado seguindo estritamente a sequência de 4 etapas exigida:

```text
┌────────────────────────┐     ┌─────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│    Schedule Trigger    │ ──> │      RSS Read       │ ──> │  Code (Top 3 Digest) │ ──> │      Send Email      │
│ (Disparo Diário 09:00) │     │ (Feed RSS G1 Tech)  │     │(Filtra 3 & Consolida)│     │ (1 E-mail com as 3)  │
└────────────────────────┘     └─────────────────────┘     └──────────────────────┘     └──────────────────────┘
```

> **Por que utilizar o nó Code na etapa 3?**  
> No n8n, nós executam para cada item da lista recebida. Se utilizássemos apenas um nó `Limit` simples repassando 3 itens individuais, o nó `Send Email` dispararia **3 e-mails separados** (um para cada notícia). Com o nó **Code** (JavaScript - Seção 3.7 da apostila), o array de notícias é limitado a 3 itens e consolidado em um **único payload enriquecido**, garantindo o envio de **exatamente 1 e-mail corporativo** contendo o resumo das 3 matérias completas.

---

## 📋 Mapeamento e Atendimento aos Requisitos

| Requisito | Descrição Oficial | Nó no n8n | Implementação no Projeto |
| :---: | :--- | :---: | :--- |
| **R1** | Disparo automático no intervalo desejado (ex: diariamente às 09:00). | `Schedule Trigger` | Configurado para disparo diário às `09:00` no fuso horário `America/Sao_Paulo` (prevenindo divergência UTC). |
| **R2** | Requisição HTTP e extração de feed RSS público fornecido. | `RSS Read` | Consome o feed do G1 Tecnologia (`https://g1.globo.com/rss/g1/tecnologia/`), estruturando objetos com `title`, `link`, etc. |
| **R3** | Processar a lista de artigos e manter apenas as 3 notícias mais recentes. | `Code` (JavaScript) | Aplica a regra de limitação (`$input.all().slice(0, 3)`), evitando excesso de envios e preparando o boletim consolidado. |
| **R4** | Formatação dinâmica injetando variáveis (`{{ $json["title"] }}` e `{{ $json["link"] }}`). | `Code` + `Send Email` | Constrói dinamicamente a mensagem com os títulos e links clicáveis das 3 notícias em HTML responsivo e texto puro. |

---

## 📂 Estrutura dos Arquivos do Repositório

```text
workflow-n8n-noticias-tech/
├── .gitignore                          # Arquivos ignorados pelo Git
├── README.md                           # Documentação acadêmica e técnica do projeto
└── workflow_noticias_tech.json         # Workflow pronto para importação no n8n
```

---

## ⚙️ Detalhamento dos Nós

### 1. Schedule Trigger (Gatilho Temporal)
- **Tipo:** `n8n-nodes-base.scheduleTrigger` (v1.2)
- **Intervalo:** *Days* (Diário)
- **Horário:** `09:00`
- **Fuso Horário:** `America/Sao_Paulo`

### 2. RSS Read (Consumo do Feed)
- **Tipo:** `n8n-nodes-base.rssFeedRead` (v1.1)
- **URL Padrão:** `https://g1.globo.com/rss/g1/tecnologia/`
- **URLs Alternativas Homologadas:**
  - UOL Tecnologia: `https://rss.uol.com.br/feed/tecnologia.xml`
  - The Guardian Tech: `https://www.theguardian.com/us/technology/rss`
  - Reddit Technology Top: `https://www.reddit.com/r/technology/top.rss?t=day`

### 3. Code - Limit & Consolida Top 3 (JavaScript)
- **Tipo:** `n8n-nodes-base.code` (v2)
- **Modo:** *Run Once for All Items*
- **Algoritmo Aplicado (conforme Seção 3.7 da apostila):**
```javascript
// Extrai as 3 notícias mais recentes e consolida o relatório em 1 único e-mail
const todosOsItens = $input.all();
const top3 = todosOsItens.slice(0, 3);

let itensHtml = '';
let itensTexto = '';

top3.forEach((item, index) => {
  const titulo = item.json.title || 'Sem título';
  const link = item.json.link || '#';
  const pubDate = item.json.pubDate ? new Date(item.json.pubDate).toLocaleString('pt-BR') : 'Recente';
  
  itensHtml += `
    <div style="margin-bottom: 16px; padding: 14px 18px; border-left: 4px solid #0b57d0; background-color: #ffffff; border-radius: 4px; box-shadow: 0 1px 3px rgba(0,0,0,0.06);">
      <h3 style="margin: 0 0 6px 0; font-size: 16px; color: #1a73e8;">
        ${index + 1}. ${titulo}
      </h3>
      <p style="margin: 0 0 10px 0; font-size: 12px; color: #666;">📅 Publicado em: ${pubDate}</p>
      <a href="${link}" target="_blank" style="display: inline-block; padding: 8px 14px; background-color: #0b57d0; color: #ffffff; text-decoration: none; border-radius: 4px; font-size: 13px; font-weight: bold;">
        Acessar Notícia Completa ➔
      </a>
    </div>
  `;
  
  itensTexto += `${index + 1}. ${titulo}\nLink: ${link}\nData: ${pubDate}\n\n`;
});

const hoje = new Date().toLocaleDateString('pt-BR');
const assunto = `[Tech News] Boletim Diário - Top 3 Notícias (${hoje})`;

const corpoHtml = `
  <div style="font-family: Arial, sans-serif; max-width: 650px; margin: 0 auto; padding: 25px; background-color: #f8f9fa; border: 1px solid #e0e0e0; border-radius: 8px;">
    <div style="background-color: #0b57d0; padding: 16px; border-radius: 6px; color: #ffffff; text-align: center;">
      <h2 style="margin: 0; font-size: 20px;">📰 Relatório Diário de Notícias Tech</h2>
      <p style="margin: 5px 0 0 0; font-size: 13px; opacity: 0.9;">3 Notícias Mais Recentes | ${hoje}</p>
    </div>
    <div style="padding: 20px 0;">
      ${itensHtml}
    </div>
    <hr style="border: none; border-top: 1px solid #ddd; margin: 20px 0;" />
    <div style="text-align: center; color: #777; font-size: 12px;">
      <p style="margin: 0;">Pipeline Autônomo n8n & RSS | Tópicos Especiais em Programação</p>
      <p style="margin: 4px 0 0 0;">IFPI Campus Piripiri - TADS</p>
    </div>
  </div>
`;

const corpoTexto = `Relatório Diário de Notícias Tech - Top 3 (${hoje})\n\n${itensTexto}---\nPipeline Autônomo n8n | TEP - IFPI Campus Piripiri`;

return [
  {
    json: {
      subject: assunto,
      text: corpoTexto,
      html: corpoHtml,
      totalNews: top3.length,
      date: hoje
    }
  }
];
```

### 4. Send Email (Disparo da Notificação)
- **Tipo:** `n8n-nodes-base.emailSend` (v2.1)
- **Assunto:** `={{ $json["subject"] }}`
- **Corpo em Texto:** `={{ $json["text"] }}`
- **Corpo em HTML:** `={{ $json["html"] }}`
- **Resultado:** Dispara exatamente **1 e-mail** contendo a compilação das 3 matérias formatadas.

---

## 🚀 Como Executar o Projeto no n8n

1. **Acessar o n8n:** Conecte-se à sua instância local ou Cloud do n8n.
2. **Importar o Workflow:**
   - No Canvas do n8n, abra o arquivo [`workflow_noticias_tech.json`](./workflow_noticias_tech.json), copie todo o seu conteúdo JSON e pressione `Ctrl + V` na área de trabalho do n8n (ou use o menu `... > Import from File...`).
3. **Configuração de Credenciais:**
   - No nó **Send Email**, adicione ou selecione suas credenciais SMTP (ex: Gmail via [Senha de App](https://myaccount.google.com/apppasswords), Mailtrap ou Outlook).
   - Preencha os campos `From Email` e `To Email` com seus endereços correspondentes.
4. **Execução de Teste:**
   - Clique em **Test workflow** na barra inferior do Canvas. O fluxo será executado pontualmente e 1 único e-mail com as 3 notícias será entregue na sua caixa de entrada.
5. **Ativação:**
   - Ative a chave seletora **Active** no canto superior direito para manter o agendamento diário automático.

---

## 📚 Referências

- **SAMPAIO, Maykol.** *Apostila de Automação Web e Python: Unidade III - Orquestração com n8n*. Disponível em: [https://maykolsampaio.github.io/cursos-maykol/cursos/automacao-web-python/unidade-3/unidade-3/](https://maykolsampaio.github.io/cursos-maykol/cursos/automacao-web-python/unidade-3/unidade-3/).
- **n8n DOCUMENTATION.** *Code Node & RSS Feed Read*. Disponível em: [https://docs.n8n.io/](https://docs.n8n.io/).
