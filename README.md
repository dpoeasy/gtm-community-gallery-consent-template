# DPOEasy — Modelo de Consentimento para Google Tag Manager

Modelo (template) do Google Tag Manager que integra o banner de cookies DPOEasy
ao [Google Consent Mode v2](https://developers.google.com/tag-platform/security/guides/consent),
permitindo que tags do Google e de terceiros ajustem seu comportamento conforme
o consentimento do visitante.

## O que ele faz

1. Define o estado padrão de consentimento como **negado** para todas as
   categorias não essenciais, antes de qualquer tag disparar.
2. Lê o cookie `DPOEasyUserConsent`, gravado pelo banner DPOEasy.
3. Aplica as escolhas já registradas pelo visitante via `updateConsentState`.

Tags marcadas com verificação de consentimento (por exemplo, uma tag de
analytics que exige `analytics_storage`) só disparam quando o visitante
autoriza a categoria correspondente.

## Instalação

1. No Gerenciador de Tags, vá em **Modelos** → **Galeria da comunidade** →
   pesquise por **DPOEasy** → **Adicionar ao espaço de trabalho**.
2. Crie uma tag do tipo **DPOEasy — Consentimento do Usuário**.
3. Em acionadores, selecione **Consent Initialization - All Pages**.

> **Importante:** o acionador precisa ser o *Consent Initialization*, não o
> *All Pages*. O Consent Initialization executa antes de todos os outros
> eventos, que é o que garante o estado padrão negado a tempo.

4. Publique o contêiner.

O script do banner DPOEasy deve continuar carregando normalmente no site (ou
via uma tag de HTML personalizado no próprio GTM). O modelo não substitui o
banner — ele apenas comunica ao Google as escolhas que o banner registra.

## Configuração

### Estados padrão por região (opcional)

Por padrão, o modelo nega todas as categorias não essenciais em qualquer
região. Não é necessário configurar nada para que isso aconteça.

Use a tabela **Estados padrão** apenas quando precisar de um comportamento
diferente por região:

| Campo | Descrição | Exemplo |
|---|---|---|
| `region` | Códigos de região ISO 3166-2, separados por vírgula. Vazio = todas as regiões. | `BR, PT` |
| `granted` | Tipos de consentimento concedidos por padrão nessa região. | `security_storage` |
| `denied` | Tipos de consentimento negados por padrão nessa região. | `ad_storage, analytics_storage` |

Cada linha vira uma chamada de `setDefaultConsentState` com
`wait_for_update: 500`, que dá tempo ao banner de informar as escolhas do
visitante antes das tags dispararem.

> Uma linha com `region` vazio sobrescreve o padrão global do modelo. Se você
> quiser apenas restringir mais em uma região específica, deixe o padrão global
> intocado e adicione somente a linha da região.

### Redação de dados de anúncios e passagem de URL

| Campo | Descrição |
|---|---|
| `ads_data_redaction` | Quando ativo, redige dados de anúncios enquanto `ad_storage` estiver negado. |
| `url_passthrough` | Quando ativo, repassa informações de clique via parâmetros de URL enquanto o consentimento estiver negado. |

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

Cada categoria é um array com os nomes dos cookies consentidos. O banner grava
a mesma categoria sob todos os seus aliases (`analytics` e `analytic`,
`advertising` e `marketing`, `unknown` e `others`).

A resolução do consentimento segue a mesma regra do banner:

| `consents.mode` | Comportamento |
|---|---|
| `accept_all` | Todas as categorias concedidas. |
| `reject_non_essential` | Apenas `necessary`. |
| customizado | Categoria concedida quando seu array tem ao menos um item. |

No modo customizado, array vazio significa categoria não consentida. O banner
só exibe categorias com cookies localizados pelo scanner, então um array vazio
nunca é ambíguo.

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
