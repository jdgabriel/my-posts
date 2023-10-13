# [Pt. 2] Melhorando sua lógica com Specification Pattern

## Introdução

Estamos desenvolvendo uma aplicação para verificar os sintomas da COVID-19 dos pacientes, e designá-los para a área correta através do **Specification Pattern**

Este post contem duas partes:

- [[Pt. 1] Melhorando sua lógica com Specification Pattern](https://www.tabnews.com.br/jdgabriel/pt-1-melhorando-sua-logica-com-specification-pattern)
- **[Pt. 2] Melhorando sua lógica com Specification Pattern**

As regras básicas de negócio ja foram atendidas, e ao final do primeira parte, ficamos com a seguinte estrutura de pastas e arquivos.

```bash
# Estrutura inicial de pastas
├── src
│   ├── patient
|   │   ├── patient.entity.ts
|   │   ├── symptoms.enum.ts
|   │   ├── patient.specification.ts
│   ├── specification
|   │   ├── specification.class.ts
|   │   ├── specification.interface.ts
├── package.json
├── pnpm-lock.yaml
└── .gitignore
```

## Continuando...

Para continuarmos, precisar compor as regras existentes para criar as regras mais complexas. Que são estas:

- **O paciente precisa de UTI**

  - O Paciente deve possuir todos os sintomas comuns
  - O Paciente deve possuir pelo menos um sintoma crítico

- **O paciente precisa de avaliação médica**
  - O Paciente deve possuir todos os sintomas comuns
  - O Paciente deve possuir todos os sintomas menos comuns
  - O Paciente deve possuir pelo menos um sintoma crítico

Para que isso seja possível, precisamos aprimorar a classe de `Specification` e a interface de `ISpecification` para que aceite o método `and` e `or`.

Adicione o método abstrato para a especificação `and` e `or`, que receba uma propriedade `ISpecification`

```ts
/* src/specification/specification.interface.ts */

export abstract class ISpecification<T> {
  abstract isSatisfiedBy(target: T): boolean;
  abstract and(other: ISpecification<T>): ISpecification<T>;
  abstract or(other: ISpecification<T>): ISpecification<T>;
}
```

Adicione e implemente o método recém criado, ficará assim:

```ts
/* src/specification/specification.class.ts */

export abstract class Specification<T> implements ISpecification<T> {
  abstract isSatisfiedBy(target: T): boolean;

  and(other: ISpecification<T>): ISpecification<T> {
    return new AndSpecification<T>(this, other);
  }
  or(other: ISpecification<T>): ISpecification<T> {
    return new OrSpecification<T>(this, other);
  }
}
```

Vamos agora criar as classes que implementam `AndSpecification` e `OrSpecification`, para disponibiliza-los globalmente para as classes que estendem `Specification`.<br/>

> Você pode usar como base a Tabela da Verdade ([Table of Truth](https://en.wikipedia.org/wiki/Truth_table)) para criar outras lógicas de comparação.

## Implementação das classes

### AndSpecification

```ts
/* src/specification/specification.class.ts */

export class AndSpecification<T> extends Specification<T> {
  constructor(private readonly one: ISpecification<T>, private readonly other: ISpecification<T>) {
    super();
  }

  isSatisfiedBy(target: T): boolean {
    return this.one.isSatisfiedBy(target) && this.other.isSatisfiedBy(target);
  }
}
```

<details>
<summary>Detalhes da classe</summary>
Estender da classe primária `Specification`.

```ts
export class AndSpecification<T> extends Specification<T>
```

Indicação das especificações para comparação no método `isSatisfiedBy`, iniciando a classe principal através do método `super()`

```ts
constructor(
    private readonly one: ISpecification<T>,
    private readonly other: ISpecification<T>
) {
  super();
}
```

Método que implementa a lógica de comparação para satisfazer a especificação "and" (**&&**)

```ts
isSatisfiedBy(target: T): boolean {
  return this.one.isSatisfiedBy(target) && this.other.isSatisfiedBy(target);
}
```

</details>

### OrSpecification

```ts
/* src/specification/specification.class.ts */

export class OrSpecification<T> extends Specification<T> {
  constructor(private readonly one: ISpecification<T>, private readonly other: ISpecification<T>) {
    super();
  }

  isSatisfiedBy(target: T): boolean {
    return this.one.isSatisfiedBy(target) || this.other.isSatisfiedBy(target);
  }
}
```

<details>
<summary>Detalhes da classe</summary>
Estender da classe primária `Specification`.

```ts
export class OrSpecification<T> extends Specification<T>
```

Indicação das especificações para comparação no método `isSatisfiedBy`, iniciando a classe principal através do método `super()`

```ts
constructor(
    private readonly one: ISpecification<T>,
    private readonly other: ISpecification<T>
) {
  super();
}
```

Método que implementa a lógica de comparação para satisfazer a especificação "or" ( **||** )

```ts
isSatisfiedBy(target: T): boolean {
  return this.one.isSatisfiedBy(target) || this.other.isSatisfiedBy(target);
}
```

</details>

## Regras de negócio compostas

Com as especificações de lógicas criadas, podemos criar as duas lógicas restantes.

- Paciente precisa de UTI
- Paciente precisa de avaliação médica

### Paciente precisa de UTI

Para que essa regra de atendida, precisamos que outras regras sejam atendidas. Que são:

- O paciente deve possuir todos os sintomas comuns
  - Regra criado em: **HasAllCommonSymptomsSpecification**
- O Paciente deve possuir pelo menos um sintoma crítico
  - Regra criado em: **HasSomeSeriousSymptomsSpecification**

Agora, vamos criar uma classe chamada `NeedsUTI`, e aplicar a lógica `AND`. Seu código ficará dessa forma.

```ts
/* src/patient/patient.specification.ts */

export class NeedsUTI extends Specification<Patient> {
  isSatisfiedBy(patient: Patient): boolean {
    return new HasAllCommonSymptomsSpecification()
      .and(new HasSomeSeriousSymptomsSpecification())
      .isSatisfiedBy(patient);
  }
}
```

Note que estamos seguinte do as regras de negócio estabelecidas na aplicação.

> O Paciente deve conter **todos os sintomas comuns** e **pelo menos um sintoma crítico** para ir para a UTI.

### Paciente precisa de atendimento médico

Para que essa regra de atendida, precisamos que outras regras sejam atendidas. Que são:

- O paciente deve possuir todos os sintomas comuns;
  - Regra criado em: **HasAllCommonSymptomsSpecification**
- O Paciente deve possuir pelo menos um sintoma crítico;
  - Regra criado em: **HasSomeSeriousSymptomsSpecification**
- O Paciente deve possuir pelo menos um sintoma menos comum;
  - Regra criado em: **HasSomeLessSymptomsSpecification**

Agora, vamos criar uma classe chamada `NeedMedical`, e aplicar a lógica `OR`. Seu código ficará dessa forma.

```ts
/* src/patient/patient.specification.ts */

export class NeedMedical extends Specification<Patient> {
  isSatisfiedBy(patient: Patient): boolean {
    return new HasAllCommonSymptomsSpecification()
      .or(new HasSomeSeriousSymptomsSpecification())
      .or(new HasSomeLessSymptomsSpecification())
      .isSatisfiedBy(patient);
  }
}
```

> O Paciente deve conter **todos os sintomas comuns** ou **pelo menos um sintoma crítico** ou **pelo menos um sintoma menos comum** para ir para o atendimento médico.

```ts
const patient = new Patient({
  name: "Jhon Doe",
  symptoms: ["PERDA_DE_FALA"],
});

const patientNeedMedical = new NeedMedical().isSatisfiedBy(patient);

if (patientNeedMedical) {
  // Sua lógica
}
```

Agora a com a base e a lógica entendidas, podemos criar qualquer lógica para atender uma regra de negócio.
Por exemplo:

> O Paciente deve **possuir sintomas comuns** e **um sintoma menos comum** e **não possuir nenhum sintomas críticos**

```ts
/* src/patient/patient.specification.ts */

export class LessUrgent extends Specification<Patient> {
  isSatisfiedBy(patient: Patient): boolean {
    return new HasAllCommonSymptomsSpecification()
      .and(new HasSomeLessSymptomsSpecification())
      .not(new HasSomeSeriousSymptomsSpecification()) // Lógica não implementada
      .isSatisfiedBy(patient);
  }
}
```

Implemente a lógica **NOT** e faça as combinações que desejar e precisar.

## Conclusão

Sempre que possível a utilização de um pattern para resolver um problema de lógica que se repete, ou que ficou muito complexa, é sempre bem vinda. O `Specification Pattern` é sempre bom quando se tem regras de negócios que são compostas por outras regras de negócio. Mas lembre-se, podemos sempre cair na `otimização prematura` da aplicação. Por isso, tenha cuidado ao utilizar patterns.
