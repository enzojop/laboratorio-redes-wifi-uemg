# Contribuindo para o Projet

## Fluxo sugerido (4 membros)

- Use branches por funcionalidade (ex.: `feat/scripts`, `docs/relatorio`).
- Abra PRs curtas e objetivas; solicite 1 revisor.
- Commits descritivos (Português, imperativo curto).

## Padrões

- Não incluir PCAP raw no repositório (apenas exemplos .example).
- Scripts com `set -euo pipefail` e comentários de ajustes (IFACE/IP).
- Documentação em Português técnico, clara e reprodutível.

## Tarefas típicas

- Atualizar configs (`configs/*`) conforme hardware
- Melhorar scripts e checagens
- Aprimorar docs e exemplos
- Criar issues para pendências e bugs

## Como abrir PR

1. Crie branch a partir de `main`.
2. Faça commits atômicos.
3. Abra PR com descrição do que mudou e teste manual.
4. Após aprovação, squash & merge.
