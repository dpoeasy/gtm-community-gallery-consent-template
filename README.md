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

### Estado padrão

Não há nada a configurar. O modelo nega todas as categorias não essenciais em
qualquer região, sempre, e **não expõe nenhuma opção capaz de afrouxar esse
comportamento** — não existe configuração que faça o consentimento falhar
aberto.

A chamada usa `wait_for_update: 500`, que dá 500 ms para o banner informar as
escolhas do visitante antes das tags dispararem.

Se o seu caso exige estado padrão diferente por país — por exemplo liberar
analytics onde não se exige consentimento prévio — abra uma issue descrevendo o
cenário. A funcionalidade foi deliberadamente deixada de fora desta versão para
que não exista configuração capaz de enfraquecer a proteção por engano.

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

Não há permissão de envio de pixel, de requisição HTTP, de acesso ao
`localStorage`, nem de leitura de variáveis globais. Você pode conferir na aba
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
