# Capítulo 9.5 — Tradução Automática e Conjunto de Dados

Este repositório/notebook implementa os passos do capítulo **9.5** do *Dive into Deep Learning* (versão PyTorch) para trabalhar com tradução automática (EN→FR).  

---

## Estrutura do Código

Abaixo está um resumo das principais funções usadas:

- `read_data_nmt()` → baixa e lê o dataset inglês-francês.  
- `preprocess_nmt(text)` → normaliza o texto (caixa baixa, espaçamento, pontuação).  
- `tokenize_nmt(text, num_examples=None)` → tokeniza em **nível de palavra**.  
- `truncate_pad(line, num_steps, padding_token)` → corta ou preenche sequências.  
- `build_array_nmt(lines, vocab, num_steps)` → converte tokens em tensores com `<eos>`, `<pad>`, etc.  
- `load_data_nmt(batch_size, num_steps, num_examples=600)` → junta tudo e retorna os iteradores + vocabulários.  

> **Nota:** foram feitos experimentos variando `num_examples`, `batch_size` e `num_steps` para avaliar impacto em vocabulários, batches e padding.

---

## Exercícios

### 1) Tente valores diferentes do argumento `num_examples` em `load_data_nmt`.  
**Como isso afeta os tamanhos do vocabulário do idioma de origem e do idioma de destino?**

**Resposta:**  
O parâmetro que realmente altera o tamanho dos vocabulários é o **`num_examples`**.  
- **Efeito direto:** quanto mais pares de tradução são usados, **maiores ficam os vocabulários** de inglês (EN) e francês (FR).  
- **Crescimento sublinear:** o aumento acontece com **ganhos marginais decrescentes** (lei de Heaps). No início, muitos tokens raros são mapeados para `<unk>`; conforme mais pares entram, parte deles atinge frequência mínima (`min_freq=2`) e passa a ser incorporada; depois, o ritmo de novidades cai.  
- **Diferença EN vs FR:** o vocabulário de francês supera o inglês logo (~300 pares) e a diferença cresce com mais exemplos. Exemplo dos seus resultados:  
  - 600 pares → EN=184 vs FR=201  
  - 2001 pares → EN=454 vs FR=585  
  - 5001 pares → EN=875 vs FR=1231  
  Isso se explica pela **morfologia mais rica e contrações/apóstrofos** no francês.  
- **Outros parâmetros:** `batch_size` e `num_steps` **não mudam vocabulário**, apenas logística: nº de batches e % de padding.  
  - Como a média de tokens por sentença fica em ~3,6–5, aumentar `num_steps` (8→12→20) eleva o padding:  
    - 5k pares: pad ~43%/38% (8 steps), ~62%/59% (12 steps), ~77%/75% (20 steps).  
- **Detalhe técnico:** o valor `pairs_used ≈ num_examples + 1` decorre do loop que começa em `i=0` e para em `i > num_examples`.  
- **Default:** se não passar `num_examples`, a função usa o valor **600**.  

---

### 2) O texto em alguns idiomas, como chinês e japonês, não tem indicadores de limite de palavras (ex.: espaço).  
**A tokenização em nível de palavra ainda é uma boa ideia para esses casos? Por que ou por que não?**

**Resposta:**  
**Não é a melhor escolha por padrão.**  

- **Problema central:** como não há separadores naturais, é preciso usar **segmentadores externos** (jieba, MeCab, Sudachi etc.). Isso traz **erros de segmentação**, **inconsistências entre dicionários/domínios**, aumento de OOVs e vocabulários inchados.  
- **Impacto:** inflar vocabulários com muitas formas pouco reaproveitáveis e contaminar treino/validação/teste com ruído.  
- **Alternativas modernas:**  
  - **Subpalavras** (BPE, WordPiece, SentencePiece-Unigram): funcionam direto no texto cru, reduzem OOVs e mantêm vocabulários compactos. SentencePiece é especialmente útil porque treina direto de texto sem espaço.  
  - **Nível de caractere/byte:** robusto contra OOV e simples de implementar, ainda que produza sequências mais longas.  
- **Exceção:** tokenização por “palavra” só faz sentido se a tarefa exigir fronteiras lexicais explícitas e houver um segmentador altamente consistente aplicado em todo o pipeline — mesmo assim, é uma solução frágil em comparação a subpalavras ou caracteres.  

---

## Conclusão

- **`num_examples`** → controla diretamente o tamanho dos vocabulários; mais exemplos = vocabulários maiores (com crescimento sublinear).  
- **Francês > Inglês** → por morfologia e contrações.  
- **`batch_size` e `num_steps`** → não alteram vocabulário, só batches/padding.  
- **Idiomas sem espaço** → tokenizar por palavra não é ideal; subpalavras ou caracteres são escolhas mais robustas.
