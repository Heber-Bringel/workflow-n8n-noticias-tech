<div align="center">

# Instituto Federal do Piauí – IFPI Campus Piripiri
### Curso Superior de Tecnologia em Análise e Desenvolvimento de Sistemas (TADS)
### Disciplina: Tópicos Especiais em Programação
### Docente: Prof. Me. Maykol Sampaio
### Discente: Heber Bringel

---

## 📰 Pipeline Autônomo de Notícias Tech por E-mail (n8n & RSS)
**Unidade III: Orquestração de Automações & Arquitetura Orientada a Eventos (EDA)**

</div>

---

## 📌 Contextualização e Problema

No cenário corporativo moderno, o acompanhamento manual de atualizações em blogs, portais e documentações técnicas consome tempo e gera retrabalho. 

Esta atividade prática tem como objetivo desenvolver um **pipeline autônomo e agendado no n8n** para monitorar fontes de notícias públicas via protocolo RSS, filtrar as 3 matérias mais recentes e disparar um relatório formatado por e-mail com título e link direto para cada publicação.

---

## 🔄 Arquitetura do Workflow

O pipeline foi projetado seguindo estritamente a sequência de 4 nós exigida:

```text
┌────────────────────────┐     ┌─────────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│    Schedule Trigger    │ ──> │      RSS Read       │ ──> │     Limit / Code     │ ──> │      Send Email      │
│ (Disparo Diário 09:00) │     │ (Feed RSS G1 Tech)  │     │  (Top 3 Mais Recentes│     │(Disparo Dinâmico HTML│
└────────────────────────┘     └─────────────────────┘     └──────────────────────┘     └──────────────────────┘
```

---

## 📋 Mapeamento e Atendimento aos Requisitos

| Requisito | Descrição Oficial | Nó no n8n | Implementação no Projeto |
| :---: | :--- | :---: | :--- |
| **R1** | Disparo automático no intervalo desejado (ex: diariamente às 09:00). | `Schedule Trigger` | Configurado para disparo diário às `09:00` alinhado ao fuso `America/Sao_Paulo` (prevenindo defasagem UTC). |
| **R2** | Requisição HTTP e extração de feed RSS público fornecido. | `RSS Read` | Consome o feed de tecnologia do G1 (`https://g1.globo.com/rss/g1/tecnologia/`), estruturando objetos com `title`, `link`, etc. |
| **R3** | Processar a lista de artigos e reter apenas as 3 notícias mais recentes. | `Limit` / `Code` | Filtragem estrita de volume via nó nativo `Limit` (`maxItems: 3`) e alternativa via nó `Code` JavaScript (`$input.all().slice(0, 3)`). |
| **R4** | Formatação dinâmica injetando variáveis (`{{ $json["title"] }}` e `{{ $json["link"] }}`). | `Send Email` | Montagem dinâmica do assunto e do corpo de e-mail (HTML estilizado com botão de acesso e fallback em texto plano). |

---

## 📂 Estrutura dos Arquivos do Repositório

```text
workflow-n8n-noticias-tech/
├── .gitignore                          # Arquivos e pastas ignoradas pelo Git
├── README.md                           # Documentação acadêmica e técnica do projeto
├── workflow_noticias_tech.json         # Workflow principal (Schedule -> RSS -> Limit -> Email)
└── workflow_noticias_tech_code.json    # Workflow alternativo com nó Code JavaScript
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

### 3. Limit / Code (Filtro Top 3)
- **Versão Nativa (`workflow_noticias_tech.json`):**
  - **Nó:** `n8n-nodes-base.limit` (v1)
  - **Max Items:** `3`
  - **Keep:** `First Items`

- **Versão com JavaScript (`workflow_noticias_tech_code.json`):**
  - **Nó:** `n8n-nodes-base.code` (v2)
  - **Código Executado (conforme Seção 3.7 da apostila):**
    ```javascript
    // Extrai os itens do feed e preserva as 3 notícias mais recentes
    const todosOsItens = $input.all();
    const top3Noticias = todosOsItens.slice(0, 3);

    return top3Noticias.map((item, index) => {
      return {
        json: {
          indice: index + 1,
          title: item.json.title || "Sem título",
          link: item.json.link || "",
          pubDate: item.json.pubDate || "",
          descricao: item.json.contentSnippet || item.json.description || ""
        }
      };
    });
    ```

### 4. Send Email (Notificação Dinâmica)
- **Tipo:** `n8n-nodes-base.emailSend` (v2.1)
- **Assunto:** `=[Tech News] {{ $json["title"] }}`
- **Corpo em Texto Puro:**
  ```text
  =Notícia de Tecnologia do Dia

  {{ $json["title"] }}

  Leia a matéria completa em:
  {{ $json["link"] }}

  ---
  Enviado automaticamente pelo pipeline n8n | TEP - IFPI Campus Piripiri
  ```
- **Corpo em HTML:** Layout responsivo com card e botão com link direto para a notícia.

---

## 🚀 Como Executar o Projeto no n8n

1. **Acessar o n8n:** Conecte-se à sua instância local ou Cloud do n8n.
2. **Importar o Workflow:**
   - No Canvas do n8n, abra o arquivo [`workflow_noticias_tech.json`](./workflow_noticias_tech.json), copie todo o seu conteúdo JSON e pressione `Ctrl + V` na área de trabalho do n8n (ou clique em `... > Import from File...`).
3. **Configuração de Credenciais:**
   - No nó **Send Email**, adicione ou selecione uma credencial SMTP (ex: Gmail via Senha de App, Mailtrap ou Outlook).
   - Preencha os campos `From Email` e `To Email` com seus endereços correspondentes.
4. **Execução de Teste:**
   - Para simular o disparo imediatamente sem aguardar às 09:00, clique no nó **Schedule Trigger** e escolha **Test step** ou clique no botão **Test workflow** na barra inferior.
5. **Ativação:**
   - Ative a chave seletora **Active** no canto superior direito do n8n para manter o agendamento em segundo plano.

---

## 📚 Referências

- **SAMPAIO, Maykol.** *Apostila de Automação Web e Python: Unidade III - Orquestração com n8n*. Disponível em: [https://maykolsampaio.github.io/cursos-maykol/cursos/automacao-web-python/unidade-3/unidade-3/](https://maykolsampaio.github.io/cursos-maykol/cursos/automacao-web-python/unidade-3/unidade-3/).
- **n8n DOCUMENTATION.** *RSS Feed Read Node & Schedule Trigger*. Disponível em: [https://docs.n8n.io/](https://docs.n8n.io/).
