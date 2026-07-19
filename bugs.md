# Incident Reports — Projeto SauceDemo

Registro de defeitos encontrados durante a execução dos testes funcionais documentados em [`README_saucedemo.md`](./README_saucedemo.md). Cada incidente referencia o caso de teste que o originou.

Campos seguem a estrutura de relatório de incidente orientada pela ISO/IEC/IEEE 29119-3 e pelo ISTQB (defect report), adaptada para este projeto.

---

## INC-001 — Checkout não avança com `problem_user`

- **Caso de teste relacionado:** TC-015
- **Ambiente:** saucedemo.com
- **Pré-condição:** login com `problem_user`, produto no carrinho, dados de checkout preenchidos
- **Passos para reproduzir:** adicionar produto ao carrinho → iniciar checkout → preencher dados → avançar
- **Resultado esperado:** avança para o resumo do pedido (Overview)
- **Resultado observado:** a aplicação retorna à página do carrinho, sem avançar
- **Severidade:** Alta (bloqueia o fluxo principal para este usuário)
- **Prioridade:** Média (usuário de teste intencionalmente instável, não representa a base de usuários reais)
- **Evidência:** print de tela e vídeo

---

## INC-002 — "Add to cart" inoperante em itens específicos com `problem_user`

- **Caso de teste relacionado:** TC-015a
- **Ambiente:** saucedemo.com
- **Pré-condição:** login com `problem_user`
- **Passos para reproduzir:** acessar a vitrine → clicar em "Add to cart" em diferentes itens
- **Resultado esperado:** todos os itens são adicionados ao carrinho
- **Resultado observado:** o comando fica inoperante em determinados itens
- **Severidade:** Média
- **Prioridade:** Baixa (usuário de teste intencionalmente instável)
- **Evidência:** print de tela e vídeo

---

## INC-003 — Remoção pelo botão do item não funciona fora da página do carrinho com `problem_user`

- **Caso de teste relacionado:** TC-015b
- **Ambiente:** saucedemo.com
- **Pré-condição:** login com `problem_user`, produto adicionado ao carrinho
- **Passos para reproduzir:** na vitrine, clicar em "Remove" no item já adicionado
- **Resultado esperado:** item removido diretamente pelo botão do item
- **Resultado observado:** comando não funciona na vitrine; remoção só é possível dentro da página do carrinho
- **Severidade:** Baixa (existe caminho alternativo funcional)
- **Prioridade:** Baixa
- **Evidência:** print de tela e vídeo

---

## INC-004 — Checkout concluído com carrinho vazio (`standard_user`)

- **Caso de teste relacionado:** TC-016
- **Ambiente:** saucedemo.com
- **Pré-condição:** login com `standard_user`, carrinho sem itens
- **Passos para reproduzir:** acessar o carrinho vazio → Checkout → preencher informações → avançar até a finalização
- **Resultado esperado:** sistema impede o início do checkout sem itens no carrinho
- **Resultado observado:** checkout concluído normalmente — Order Receipt exibe 0 itens e total $0.00
- **Severidade:** Alta (viola regra de negócio central com o usuário-base da aplicação, não um usuário de teste especial)
- **Prioridade:** Alta
- **Evidência:** PDF do Order Receipt com item e valor zero
