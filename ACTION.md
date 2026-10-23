# Projeto

Primeiro é preciso entender que meus actions não estão separados em uma Action para ***Changelog*** e um para ***Release Package***, as duas funções estão juntas na mesma Action.

## Explicação (linha a linha)

### 1.
```bash
on:
  push:
    tags:
      - v[0-9]+.[0-9]+.[0-9]+
```
- Nesta linha de comando, está definindo que o workflow funcionará através de ***Git Push*** que contenham alguma TAG.

```
Exemplo : git push origin v1.1.0
```
### 2. Job : Deploy

 O ***Job*** Deploy tem diversos ***Steps*** para o funcionamento completo da Action.

 


