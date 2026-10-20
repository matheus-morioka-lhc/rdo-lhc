# rdo-lhc — Diário da Obra dos Empreiteiros (v07 · 2026-10-20)

Formulário de celular (GitHub Pages) + Apps Script na planilha **RDO - Empreiteiros (planilha compartilhada)**.
O empreiteiro preenche pelo link dele (obra × empreiteiro); a planilha grava, gera alertas, avisa o engenheiro e monta o RDO interno em PDF às 8h.

## O que mudou na v07

| Item | v06 | v07 |
|---|---|---|
| Equipamentos | Seção 4 do RDO interno sempre `[CAMPO NÃO INFORMADO]` | Empreiteiro informa equipamento, quantidade e de quem é (LHC / próprio / não sabe). Aba nova `EQUIPAMENTOS_USO`; lista na coluna `EQUIPAMENTOS` da aba LISTAS |
| Empenho | Só alerta quando o fiscal manda começar serviço novo | Colunas `ENGENHEIRO`, `EMAIL_ENGENHEIRO`, `EMPENHO`, `EMPENHO_CONFERIDO_EM` na aba OBRAS (fonte: Painel Gerencial). Gente trabalhando em obra sem `EMPENHO = SIM` → alerta CRÍTICO com cópia ao gestor |
| Aviso | Só linha na aba ALERTAS | Alerta CRÍTICO vai por e-mail na hora ao engenheiro da obra; fiscal / serviço novo / sem empenho com cópia ao `EMAIL_GESTOR`. Resumo diário às 8h por engenheiro |
| Nº do envio | `getLastRow()` — repetia se apagasse linha | Maior nº já usado + 1 |
| Colunas | Gravação por posição (coluna 27 = UUID fixo) | Gravação pelo nome do cabeçalho |
| Erro do servidor | Qualquer recusa aparecia como "Sem conexão" e ficava pendente para sempre | Página diferencia sem sinal × link inativo × data fora do prazo × sistema ocupado |
| Rascunho | Rascunho antigo sem campo novo podia quebrar | Rascunho antigo recebe os campos novos |

Alertas e nível:

| Alerta | Nível | E-mail | Cópia ao gestor |
|---|---|---|---|
| Orientação verbal do fiscal | CRÍTICO | sim | sim |
| Ordem para começar serviço novo | CRÍTICO | sim | sim |
| Execução em obra sem empenho confirmado | CRÍTICO | sim | sim |
| Dia/turno impraticável | CRÍTICO | sim | não |
| Intercorrência com acidente, dano, vazamento ou furto | CRÍTICO | sim | não |
| Outras intercorrências, faltando, envio atrasado, fora do cadastro, empenho sem conferência recente | ATENÇÃO | só no resumo das 8h | não |

## Implantação (nesta ordem)

1. **Apps Script** (Extensões > Apps Script na planilha): substituir todo o `Code.gs` pelo desta versão. Salvar.
2. **Planilha** → menu `RDO EMPREITEIROS > 1. Configurar planilha`. Cria `EQUIPAMENTOS_USO`, acrescenta as colunas novas no fim de OBRAS, ENVIOS e ALERTAS, a coluna EQUIPAMENTOS em LISTAS e os parâmetros novos em CONFIG. Não apaga nem move nada.
3. **CONFIG**: preencher `EMAIL_GESTOR` (Eng. Gabriel) — está `[A CONFIRMAR]`. Conferir `RODAPE` com o cartão CNPJ.
4. **OBRAS**: preencher as 4 colunas novas (tabela abaixo).
5. Menu `5. Testar e-mail` → autorizar o envio de e-mail (pede permissão nova na 1ª vez).
6. **Reimplantar o Web App sem trocar a URL**: Implantar > Gerenciar implantações > lápis > Versão: **Nova versão** > Implantar. (Nova implantação muda a URL e quebra os links já enviados.)
7. **GitHub**: substituir `index.html` no repositório `rdo-lhc`. O GitHub Pages atualiza em 1–2 min. Abrir o link de teste: o rodapé do formulário deve mostrar `v07`.
8. Menu `4. Ativar RDO interno diário` de novo (recria o acionador; agora inclui o resumo por e-mail).
9. Teste interno: um envio pelo link do 3221 com fiscal = sim / pediu de boca = sim → conferir e-mail, aba ALERTAS (colunas `AVISO_PARA`, `AVISO_ENVIADO_EM`) e aba `EQUIPAMENTOS_USO`. **Apagar as linhas de teste** antes de soltar para os empreiteiros.
10. Menu `2. Gerar links que faltam` para os demais pares obra × empreiteiro (hoje só existe 1 link: 3221 × GROUP SOCIPREV).

## Aba OBRAS — valores do Painel Gerencial (lidos em 20/10/2026)

| COD_OBRA | ENGENHEIRO | EMAIL_ENGENHEIRO | EMPENHO | EMPENHO_CONFERIDO_EM |
|---|---|---|---|---|
| 3221 | MATHEUS | [A CONFIRMAR] | SIM | 2026-10-20 |
| 3004 | GUILHERME | guilherme@lhceng.com | SIM | 2026-10-20 |
| 3005 | GUILHERME | guilherme@lhceng.com | SIM | 2026-10-20 |
| 3007 | GUILHERME | guilherme@lhceng.com | SIM | 2026-10-20 |
| 3008 | GUILHERME | guilherme@lhceng.com | SIM | 2026-10-20 |
| 3009 | GUILHERME | guilherme@lhceng.com | SIM | 2026-10-20 |
| 3016 | MATHEUS | [A CONFIRMAR] | SIM | 2026-10-20 |

Origem: `ENGENHEIRO` e `EMPENHO` = coleção `obras` do Painel Gerencial; e-mail do Guilherme = cadastro de engenheiros do Painel de Locações. `EMPENHO_CONFERIDO_EM` = data da leitura do painel, não data do empenho. O painel informa SIM/NÃO por obra, sem nº de empenho nem valor.

## Ligação com os painéis

- **Painel Gerencial → RDO**: só cadastro (engenheiro e empenho SIM/NÃO). Nenhum valor financeiro entra na planilha nem chega ao empreiteiro. O Apps Script não lê o banco do painel; a atualização das 4 colunas é por cópia (manual ou tarefa agendada). Se ficar mais de `DIAS_EMPENHO_VALIDO` dias sem atualizar, o envio gera alerta de atenção.
- **RDO → Painel de Locações** (próxima etapa): o painel lê a aba `EQUIPAMENTOS_USO` pelo conector do Google Drive e cruza, por CC e data, alugado em campo × declarado em uso. Chave: `COD_OBRA` = campo `cc` da obra no painel.

## Abas

`CONFIG` · `OBRAS` · `EMPREITEIROS` · `LINKS` · `LISTAS` · `ENVIOS` · `EFETIVO` · `ATIVIDADES` · `PARALISACOES` · `EQUIPAMENTOS_USO` · `ALERTAS`

Limite do Google: conta Workspace envia até 1.500 e-mails/dia pelo Apps Script; o menu 5 mostra a cota restante.
