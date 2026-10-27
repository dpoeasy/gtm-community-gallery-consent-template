# DPOEasy — Modelo de Consentimento para Google Tag Manager

Modelo (template) do Google Tag Manager que integra o banner de cookies DPOEasy
ao [Google Consent Mode v2](https://developers.google.com/tag-platform/security/guides/consent),
permitindo que tags do Google e de terceiros ajustem seu comportamento conforme
o consentimento do visitante.

## O que ele faz

1. Define o estado padrão de consentimento. Se o banner DPOEasy já mandou o
   sinal inicial dele, o modelo segue esse sinal. Se não, nega todas as
   categorias não essenciais antes de qualquer tag disparar.
2. Lê o cookie `DPOEasyUserConsent`, gravado pelo banner DPOEasy.
3. Aplica as escolhas já registradas pelo visitante via `updateConsentState`.

Tags marcadas com verificação de consentimento (por exemplo, uma tag de
analytics que exige `analytics_storage`) só disparam quando o visitante
autoriza a categoria correspondente.

## Instalação

Há duas formas de colocar o modelo no contêiner. Escolha uma e siga para o
passo 2.

1. Adicione o modelo ao espaço de trabalho:
   - **Pela Galeria da comunidade:** no Gerenciador de Tags, vá em **Modelos**
     → **Galeria da comunidade** → pesquise por **DPOEasy** → **Adicionar ao
     espaço de trabalho**.
   - **Pela importação do arquivo:** baixe o `template.tpl` deste repositório
     (ou o arquivo `.tpl` distribuído pela DPOEasy). No Gerenciador de Tags, vá
     em **Modelos** → **Modelos de tag** → **Novo**, abra o menu **⋮** no canto
     superior direito, escolha **Importar**, selecione o arquivo e clique em
     **Salvar**.
2. Crie uma tag do tipo **DPOEasy — Consentimento do Usuário**.
3. Em acionadores, selecione **Consent Initialization - All Pages**.

> **Importante:** o acionador precisa ser o *Consent Initialization*, não o
> *All Pages*. O Consent Initialization executa antes de todos os outros
> eventos, que é o que garante o estado padrão negado a tempo.

4. Publique o contêiner.

### Atualizar um modelo importado

Quem instalou por arquivo atualiza importando o arquivo novo por cima do modelo
atual: em **Modelos** → **Modelos de tag**, abra o modelo DPOEasy, escolha
**⋮** → **Importar**, selecione o `.tpl` novo e clique em **Salvar**. As tags
que já usam o modelo continuam ligadas a ele. Abra uma delas para conferir os
campos e publique o contêiner.

O modelo não substitui o banner: ele comunica ao Google as escolhas que o
banner registra.

> **Instale o script do banner no HTML, antes do snippet do GTM.** Assim o
> sinal inicial configurado no DPOEasy chega ao Google antes das tags e o
> modelo o respeita. Com o banner carregado por uma tag de HTML personalizado
> no próprio GTM, o modelo nega todas as categorias não essenciais até o
> banner carregar, e as tags de *All Pages* rodam com esse negado.

## Configuração

### Estado padrão

Não há nada a configurar no modelo. O sinal inicial é configurado no banner,
na tela do DPOEasy: cada categoria (funcionalidade, analytics, publicidade)
começa como "Negado" ou "Concedido".

- **Banner no HTML antes do snippet do GTM:** o banner manda esse sinal e
  grava a marca `window.dpoeasyConsentDefaultSent`. O modelo lê a marca e não
  aplica o próprio estado padrão por cima.
- **Sem a marca** (banner carregado por tag do GTM, ou ausente): o modelo nega
  todas as categorias não essenciais em qualquer região, com
  `wait_for_update: 500`, que dá 500 ms para o banner informar as escolhas do
  visitante antes das tags dispararem.

O modelo em si não expõe nenhuma opção capaz de afrouxar o estado padrão. O que
fica concedido no início é o que a organização configurou no DPOEasy.

### Redação de dados de anúncios e passagem de URL

| Campo | Descrição |
|---|---|
| `ads_data_redaction` | Quando ativo, redige dados de anúncios enquanto `ad_storage` estiver negado. |
| `url_passthrough` | Quando ativo, repassa informações de clique via parâmetros de URL enquanto o consentimento estiver negado. |

## Privacidade e segurança

Este modelo **não faz nenhuma requisição de rede** e **não envia dado nenhum
para fora do navegador**. Ele lê um cookie e conversa exclusivamente com a API
de consentimento do próprio Google Tag Manager.

As permissões declaradas no modelo são a prova auditável disso — o sandbox do
Tag Manager impede qualquer operação não declarada:

| Permissão | Escopo concedido |
|---|---|
| Lê cookies | Somente o cookie `DPOEasyUserConsent`. Nenhum outro |
| Acessa estado de consentimento | Leitura e escrita dos 7 tipos do Consent Mode v2 |
| Grava na camada de dados | Somente as chaves `ads_data_redaction` e `url_passthrough` |
| Registra no console | Apenas mensagens de diagnóstico quando o cookie está ilegível |
| Acessa variáveis globais | Somente leitura de `dpoeasyConsentDefaultSent`, a marca gravada pelo banner. Sem escrita nem execução |

Não há permissão de envio de pixel, de requisição HTTP, de acesso ao
`localStorage`, nem de leitura de outras variáveis globais. Você pode conferir na aba
**Permissões** do editor de modelos antes de instalar.

## Mapeamento de categorias

O banner DPOEasy trabalha com categorias de cookies. O modelo as traduz para os
tipos de consentimento do Google:

| Categoria DPOEasy | Tipos de consentimento do Google |
|---|---|
| `necessary` | `security_storage` (sempre concedido) |
| `analytics` | `analytics_storage` |
| `functionality` | `functionality_storage`, `personalization_storage` |
| `advertising` | `ad_storage`, `ad_user_data`, `ad_personalization` |
| `unknown` | — (sem equivalente no Consent Mode) |

## Contrato com o banner

O modelo lê o cookie `DPOEasyUserConsent`, no formato:

```json
{
  "consents": {
    "mode": "reject_non_essential",
    "necessary": ["DPOEasyVisitorId", "DPOEasyUserConsent"],
    "analytics": [],
    "analytic": [],
    "functionality": [],
    "advertising": [],
    "marketing": [],
    "unknown": [],
    "others": []
  },
  "selectedTags": [],
  "userConsentToken": "..."
}
```

Cada categoria é um array. No formato antigo, ele traz os nomes dos cookies
consentidos. No formato compacto, gravado pelas versões atuais do banner, traz
só a chave da categoria aceita (`["analytics"]`) ou fica vazio (`[]`). O banner
grava a mesma categoria sob todos os seus aliases (`analytics` e `analytic`,
`advertising` e `marketing`, `unknown` e `others`) e só grava as categorias que
aparecem nele.

A resolução do consentimento segue a mesma regra do banner:

| `consents.mode` | Comportamento |
|---|---|
| `accept_all` | Concede as categorias que aparecem no banner (array com itens). |
| `reject_non_essential` | Apenas `necessary`. |
| customizado | Categoria concedida quando seu array tem ao menos um item. |

Array vazio ou categoria ausente significa categoria não consentida, inclusive
no `accept_all`: o "Aceitar todos" não concede ao Google uma categoria que o
banner não mostrou ao visitante.

> **Manutenção:** a lista de categorias e aliases do modelo espelha
> `CATEGORY_DEFINITIONS` em `src/config.js` do repositório do banner. Alterações
> lá exigem alteração aqui.

## Requisitos

- Consent Mode habilitado no contêiner.
- Banner DPOEasy instalado e gravando o cookie `DPOEasyUserConsent`.

## Suporte

Abra uma [issue](https://github.com/dpoeasy/gtm-community-gallery-consent-template/issues)
neste repositório.

## Licença

Apache License 2.0. Veja [LICENSE](LICENSE).
