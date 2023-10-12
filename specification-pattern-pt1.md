# [Pt. 1] Melhorando sua lógica com Specification Pattern Introdução

## Introdução

Com a evolução da lógica do seu aplicativo é muito comum termos `if` espalhados, realizando a regra de negócio. E o que é pior, a mesma lógica espalhada em várias partes. Nesse momento que você para para pensar: Como posso simplificar a lógica das regras de negócio?
Uma ótima alternativa é usar um pattern. Nesse post irei explicar como funciona o **Specification Pattern** (Padrão de Especificação).

Este post contem duas partes:

- **[Pt. 1] Melhorando sua lógica com Spectation Pattern**
- [Pt. 2] Melhorando sua lógica com Spectation Pattern (Em desenvolvimento)

## O que é o Specification Pattern?

Specification Pattern é um padrão de design de software, onde regras de negócio podem ser recombinadas através de lógica booleana. (Wikipedia em tradução livre)

Com este pattern podemos combinar de forma simplificada uma ou mais regras para compor uma regra de negócio maior, mais clara e mais específica para satisfazer um lógica.

## Vamos ao exemplo

Nesse exemplo irei usar a COVID-19, com seus sintomas.
_(Não reflete a realidade, apenas um exemplo)_

### Regras de negócio da aplicação

#### Sintomas

- **Sintomas Comuns**
  - Febre
  - Torre seca
  - Cnasaço
- **Sintomas Menos Comuns**
  - Dores e desconfortos
  - Dor de garganta
  - Diarreia
  - Conjuntivite
  - Dor de cabeça
  - Perda de paladar ou olfato
  - Erupção cutânea
- **Sintomas Críticos**
  - Dificuldade de respirar
  - Dor no peito
  - Perda de fala

#### Especificação do Paciente

- **Precisa de UTI**
  O paciente precisa atender **todos os requisitos**. Lógica: **AND** - Possui todos os sintomas comuns - Possui pelo menos um sintoma crítico

- **Precisa de avaliação médica**
  O paciente precisa atender **um dos requisitos**. Lógica: **OR** - Possui todos os sintomas comuns - Possui todos os sintomas menos comuns - Possui pelo menos um sintoma crítico

### Mão no código

Escolha uma estrutura de pasta que melhor de adapta ao seu projeto. Para este exemplo, iremos adotar uma estrutura de pastas simples e objetiva para fins didáticos. E fica assim:

```bash
# Estrutura inicial de pastas
├── src
│   ├── patient
│   ├── specification
├── package.json
├── pnpm-lock.yaml
└── .gitignore
```

#### Criando a classe para Sintomas e Paciente

##### Sintomas

```ts
// src/patient/symptoms.enum.ts

export enum SymptomsEnum {
  "FEBRE",
  "TOSSE_SECA",
  "CANSACO",
  "DORES",
  "DOR_DE_GARGANTA",
  "DIARREIRA",
  "CONJUNTIVITE",
  "DOR_DE_CABECA",
  "PERDA_DO_PALADAR",
  "ERUPCAO_CULTANE",
  "DIFICULDADE_DE_RESPIRAR",
  "DOR_NO_PEITO",
  "PERDA_DE_FALA",
}

export type Symptom = keyof typeof SymptomsEnum;
```

##### Paciente

```ts
// src/patient/patient.entity.ts

export class Patient {
  public name: string;
  public symptoms: Symptom[];

  constructor(name: string, symptoms: Symptom[]) {
    this.name = name;
    this.symptoms = symptoms;
  }
}
```

#### Criando a interface Specification

Dentro da pasta `specification` crie um arquivo que irá conter a interface que iremos utilizar durante todo o exemplo.

> Você também pode criá-la como classe abstrata

```ts
// src/specification/specification.interface.ts

export interface ISpecification<T> {
  isSatisfiedBy(target: T): boolean;
}
```

Criamos uma interface que possui apenas uma função, que recebe um `target` e retorna um `boolean`, apenas.

#### Sua primeira especificação

Na pasta `patient` crie um arquivo de especificações: `patient.specification.ts`

### _O paciente deve possuir todos os sintomas comuns_

O código ficaria assim:

```ts
// src/patient/patient.specification.ts

import { Specification } from "../specification/specification.interface";
import { Patient } from "./patient.entity";
import { Symptom } from "./symptoms.enum";

export class HasAllCommonSymptomsSpecification extends Specification<Patient> {
  private readonly SYMPTOM_CRITERIAL_FEVER: Symptom = "FEBRE";
  private readonly SYMPTOM_CRITERIAL_TOSSE_SECA: Symptom = "TOSSE_SECA";
  private readonly SYMPTOM_CRITERIAL_CANSACO: Symptom = "CANSACO";

  isSatisfiedBy(patient: Patient): boolean {
    const symptoms = patient.symptoms;
    return (
      symptoms.includes(this.SYMPTOM_CRITERIAL_FEVER) &&
      symptoms.includes(this.SYMPTOM_CRITERIAL_TOSSE_SECA) &&
      symptoms.includes(this.SYMPTOM_CRITERIAL_CANSACO)
    );
  }
}
```

Assim aplicamos nosso primeira regra de negócio através do `Specification Pattern`. Ou seja, definimos que se o paciente conter todos os sintomas comuns, ele atenderá a especificação `HasAllCommonSymptomsSpecification`

<details>
<summary>Detalhes da especificação</summary>
  
- Importação necessárias
```ts
import { Specification } from "../specification/specification.interface";
import { Patient } from "./patient.entity";
import { Symptom } from "./symptoms.enum";
```
- Classe estendida de `Specification` interface que criamos, atribuindo o generic `Patient`
    
```ts
export class HasAllCommonSymptomsSpecification extends Specification<Patient>
```
    
- Definição das variáveis para os critérios da especificação

```ts
 private readonly SYMPTOM_CRITERIAL_FEVER: Symptom = "FEBRE";
 private readonly SYMPTOM_CRITERIAL_TOSSE_SECA: Symptom = "TOSSE_SECA";
 private readonly SYMPTOM_CRITERIAL_CANSACO: Symptom = "CANSACO";
```

- Método que define se a especificação foi atendida

```ts
isSatisfiedBy(patient: Patient): boolean {
    const symptoms = patient.symptoms;
    return (
      symptoms.includes(this.SYMPTOM_CRITERIAL_FEVER) &&
      symptoms.includes(this.SYMPTOM_CRITERIAL_TOSSE_SECA) &&
      symptoms.includes(this.SYMPTOM_CRITERIAL_CANSACO)
    );
}
```

</details>
    
<details>
    <summary>Como usar a especificação</summary> 
    
```ts
const patient = new Patient({
  name: "Jhon Doe",
  symptoms: ["FEBRE", "CANSACO", "TOSSE_SECA"],
});

const hasAllCommonsSymptoms = HasAllCommonSymptomsSpecification.isSatisfiedBy(patient);

if(hasAllCommonsSymptoms){
// Sua lógica
}

````
</details>
<hr/>

### *O paciente deve possuir pelo menos um dos sintomas menos comum*

O código ficaria assim:

```ts
// src/patient/patient.specification.ts

import { Specification } from "../specification/specification.interface";
import { Patient } from "./patient.entity";
import { Symptom } from "./symptoms.enum";

export class HasSomeLessCommonSymptomsSpecification extends Specification<Patient> {
  private readonly SYMPTOM_CRITERIAL: Symptom[] = [
    "DORES",
    "DOR_DE_GARGANTA",
    "DIARREIRA",
    "CONJUNTIVITE",
    "DOR_DE_CABECA",
    "PERDA_DO_PALADAR",
    "ERUPCAO_CULTANE",
  ];

  isSatisfiedBy(patient: Patient): boolean {
    const symptoms = patient.symptoms;
    return symptoms.map((symptom) =>
        this.SYMPTOM_CRITERIAL.includes(symptom)
    ).includes(true);
  }
}
````

Definimos que se o paciente conter pelo menos um dos sintomas menos comuns, ele atenderá a especificação `HasSomeLessCommonSymptomsSpecification`

<details>
<summary>Detalhes da especificação</summary>
  
- Importação necessárias
```ts
import { Specification } from "../specification/specification.interface";
import { Patient } from "./patient.entity";
import { Symptom } from "./symptoms.enum";
```
- Classe estendida de `Specification` interface que criamos, atribuindo o generic `Patient`
    
```ts
export class HasSomeLessCommonSymptomsSpecification extends Specification<Patient>
```
    
- Definição das variáveis para os critérios da especificação

```ts
  private readonly SYMPTOM_CRITERIAL: Symptom[] = [
    "DORES",
    "DOR_DE_GARGANTA",
    "DIARREIRA",
    "CONJUNTIVITE",
    "DOR_DE_CABECA",
    "PERDA_DO_PALADAR",
    "ERUPCAO_CULTANE",
  ];
```

- Método que define se a especificação foi atendida

```ts
  isSatisfiedBy(patient: Patient): boolean {
    const symptoms = patient.symptoms;
    return symptoms.map((symptom) =>
        this.SYMPTOM_CRITERIAL.includes(symptom)
    ).includes(true);
  }
```

</details>
    
<details>
    <summary>Como usar a especificação</summary> 
    
```ts
const patient = new Patient({
  name: "Jhon Doe",
  symptoms: ["DOR_DE_CABECA"],
});

const hasSomeLessCommonSymptom = HasSomeLessCommonSymptomsSpecification.isSatisfiedBy(patient);

if(hasSomeLessCommonSymptom){
// Sua lógica
}

````
</details>
<hr/>

### *O paciente deve possuir pelo menos um sintoma crítico*

O código ficaria assim:

```ts
// src/patient/patient.specification.ts

import { Specification } from "../specification/specification.interface";
import { Patient } from "./patient.entity";
import { Symptom } from "./symptoms.enum";

export class HasSomeSeriousSymptomsSpecification extends Specification<Patient> {
  private readonly SYMPTOM_CRITERIAL: Symptom[] = [
    "DIFICULDADE_DE_RESPIRAR",
    "DOR_NO_PEITO",
    "PERDA_DE_FALA"
  ];

  isSatisfiedBy(patient: Patient): boolean {
    const symptoms = patient.symptoms;
    return symptoms.map((symptom) =>
        this.SYMPTOM_CRITERIAL.includes(symptom)
    ).includes(true);
  }
}
````

Definimos que se o paciente conter pelo menos um dos sintomas críticos, ele atenderá a especificação `HasSomeSeriousSymptomsSpecification`

<details>
<summary>Detalhes da especificação</summary>
  
- Importação necessárias
```ts
import { Specification } from "../specification/specification.interface";
import { Patient } from "./patient.entity";
import { Symptom } from "./symptoms.enum";
```
- Classe estendida de `Specification` interface que criamos, atribuindo o generic `Patient`
    
```ts
export class HasSomeSeriousSymptomsSpecification extends Specification<Patient>
```
    
- Definição das variáveis para os critérios da especificação

```ts
  private readonly SYMPTOM_CRITERIAL: Symptom[] = [
    "DIFICULDADE_DE_RESPIRAR",
    "DOR_NO_PEITO",
    "PERDA_DE_FALA"
  ];
```

- Método que define se a especificação foi atendida

```ts
  isSatisfiedBy(patient: Patient): boolean {
    const symptoms = patient.symptoms;
    return symptoms.map((symptom) =>
        this.SYMPTOM_CRITERIAL.includes(symptom)
    ).includes(true);
  }
```

</details>
    
<details>
    <summary>Como usar a especificação</summary> 
    
```ts
const patient = new Patient({
  name: "Jhon Doe",
  symptoms: ["DOR_NO_PEITO"],
});

const hasSomeSeriousCommonSymptom =
HasSomeSeriousSymptomsSpecification.isSatisfiedBy(patient);

if(hasSomeSeriousCommonSymptom){
// Sua lógica
}

```
</details>

<hr />

Com as especificações básicas do paciente criadas, agora podemos combinar as especificações para que as regras de negócio mais complexas sejam atendidas.

### **Continuamos na parte dois...**
Iremos criar o restante das regras de negócio e criar testes automatizados para certificar que todas as regras estão sendo atendidas.


```
