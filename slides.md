---
theme: ./theme-apple-basic
title: La gestion des erreurs en JS/TS
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: none
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
layout: statement
---

# Hello ! 

👋

---
layout: image-right
image: /images/clem-madere.jpeg
---

# whoami


## Clément Ollivier

<div class="mt-10">
<p>Staff frontend engineer @ HelloAsso</p>
<p>Intervenant @ ECV Digital</p>
<p>Previously @ Webians & NuxtLabs</p>
</div>

<br />

<div>
<p>
  <mingcute-bluesky-social-fill /> @clem.codes
</p>
<p>
  <mingcute-github-fill /> @clemcode
</p>
<p>
<mingcute-linkedin-fill /> Clément Ollivier
</p>
</div>

---
layout: statement
---

# Gestion des erreurs 
# en JS/TS

<!-- 
D'où je parle
- frontend avant tout
- un peu de back avec nodeJS

En tant que staff, réflexion archi

Je me suis intéressé à notre gestion des erreurs

Talk = résumé de ma réflexion
-->

---
layout: section
---

# Le menu

<br />

## C'est quoi une erreur ?

<br />

## Représenter une erreur en JS

<br />

## Patterns de gestion d'erreurs 
### (Le 3ème va vous surprendre)

<br />

## Discussion

---
layout: section
---
# C'est quoi une erreur ?

<br />

<h2 v-click="1">
  Happy path
</h2>
<p v-click="1">Quand ça se passe normalement</p>

<h2 v-click="2">
  Sad path
</h2>
<p v-click="2">Quand ça ne se passe pas normalement</p>

<!--
Souvent quand on parle d'erreurs, on voit apparaître la notion de Happy path

Les erreurs sont le sad path

Réfléchir seulement comme ça, c'est un peu limitant

Pour gratter un peu cette notion, quizz
-->

---
layout: statement
---

# Quizz

<img alt="QR Code" src="/images/qr-code.png" style="height: 20rem; margin: 0 auto; " />

https://clemcode.github.io/happy-sad-voting/

<!--
Affichage de cas de figure rencontrés dans un parcours utilisateur ou l'éxécution d'un programme

Vote Happy path ou Sad path

-->

---
layout: statement
transition: none
---

# Transaction complétée

---
layout: statement
transition: none
---

# Syntax error

---
layout: statement
transition: none
---

# Page 404

---
layout: statement
transition: none
---

# Panier expiré

---
layout: statement
transition: none
---

# Mot de passe trop court

---
layout: statement
---

# Out of stock

---
layout: section
transition: none
---

<img src="/images/error-matrix.svg" alt="error matrix: known or unknown / stop or continue ?" style="height: 40vh; margin: 0 auto;" />

<!--
Happy / sad path = subjectif

Pour déterminer comment réagir à une situation, intéressant de faire une matrice

Axe horizontal : Comportement connu / attendu

Axe vertical : Est-ce que le traitement doit continuer ou s'arrêter
-->

---
layout: section
---

<img src="/images/error-matrix-2.svg" alt="error matrix: known or unknown / stop or continue ?" style="height: 40vh; margin: 0 auto;" />

---
layout: statement
---

## Contexte

<br />

## Expressivité

<br />

## Typage

<br />

## Prédictabilité

<!--
**Contexte** : L'erreur informe du pourquoi de son existence

Laisse le caller décider du traitement

(Une même erreur peut être critique ou non suivant les contextes)

**Expressivité** : Je veux savoir que c'est une erreur

Je veux pouvoir les repérer en scanant mon code

**Typage** : Rejoint expressivité.

Idéalement, je veux forcer le caller à gérer ces erreurs

**Prédictabilité** : Patterns identifiables et répétables


Regardons un contre exemple
-->

---
layout: split
---
<div class="mt-32">
</div>

<template v-slot:right>

  - ❌ Contexte
  - ❌ Expressivité
  - ❌ Typage
  - ❌ Prédictabilité

</template>

<template v-slot:left>

```ts
function validateUserPassword(password) {
  if (password.length < 8) {
    showNotification(
      `Votre mot de passe est trop court :(`
    )
    return PASSWORD_TOO_SHORT // string, number...
  }
}
```

</template>

<!--
**Contexte**: elle ne laisse pas le caller gérer (showNotification)

Side-effect = pas facilement testable

**Expressivité / Typage**: retourne un data type qui pourrait ne pas être une erreur

**Prédictabilité**: Je ne retrouve pas un pattern clair et reproductible
-->
---
layout: statement
---

# La classe Error

```ts
const x = new Error(message, options)

if (x instanceof Error) {
  console.log(x.message, x.cause, x.stack)
}
```

---
layout: section
transition: none
---
# Extend la classe Error

```ts twoslash
class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "ValidationError";
  }
}

const passwordTooShortError = new ValidationError("Password too short")



```

---
layout: section
---
# Extend la classe Error

```ts twoslash
class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "ValidationError";
  }
}

const passwordTooShortError = new ValidationError("Password too short")
console.log(passwordTooShortError.code)
```

---
layout: section
---
# Ajouter un comportement

```ts
class CriticalError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "Critical Error";

    logger.emit(this.name, this.cause)
  }
}
```

<!-- Comment on utilise ces erreurs concrètement dans notre programme ? 2 mécanismes, throw et return -->

---
layout: statement
---

# To throw or not to throw?

---
layout: section
---

```ts
async function connectDb() {
  // ...
  throw new CriticalError("The database is on fire")
}
```

<br />

```ts
try {
  await connectDb()
} catch(error) {
  response.send(error.message, { code: 500 })
}
```

<!--
throw = lever une exception

exception remonte la call stack jusqu'à être catch. Si pas catche, le programme se termine en état d'erreur
-->

---
layout: section
---

```ts
try {
  const { userId, payload } = JSON.parse(body)
  const db = await connectDb()
  const user = await db.find('users', userId)
  user.update(payload)
} catch(error) {
  response.send(error.message, { code: 500 })
}
```

---
layout: image-right
image: /images/clean-code.jpg
---

## Utiliser des exceptions plutôt que des codes d'erreur

<br />

## Ne pas retourner `null`

<br />

## Commencer par écrire les blocs `try` `catch`

<!--
throw / try / catch pas unqiue à JS

Le débat est tranché ? Wait

Comment collecter des erreurs ?

Java = clause throws
-->

---
layout: split
transition: none
---

<template v-slot:left>

## Et on peut typer throw ?

<br />

```ts
function connectDb() throws CriticalError {}
```
</template>

---
layout: split
---

<template v-slot:left>

## Et on peut typer throw ?

<br />

```ts
function connectDb() throws CriticalError {}
```

<br />
<h2>TL;DR Non</h2>
</template>

<template v-slot:right>
  <!-- <video alt="Long scroll d'un thread Github" class="h-[24rem]" src="/images/throw-issue.mov" autoplay loop></video> -->
</template>

---
layout: split
---

<template v-slot:left>

## Standard

<br />

## Interromps le flux

<br />

## Non typable

<br />

## Erreurs non collectables

<div v-click="1">

<br />

<hr />

<br />

## Cas d'usage

<br />

### Erreurs critiques (Unknown / Stop)

<br />

### Code consommé par des tiers

</div>

</template>

<template v-slot:right>

![Borne de freinage d'urgence](/images/emergency-stop.jpg)

<p class="text-xs">Photo de <a href="https://unsplash.com/fr/@two_tees?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Matt Walsh</a> sur <a href="https://unsplash.com/fr/photos/appareil-rond-blanc-et-rouge-3wIKwi0YGPY?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a></p>
</template>

<!-- 
Voyons voir du côté de Rust comment dont gérées les erreurs ?

-->

---
layout: section
---

<img src="/images/rust-ferris.svg" alt="rust logo" style="height: 8rem; margin-bottom: 2rem" />

```rust
use std::fs::File;

enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```

---
layout: statement
---

# Result type

---
layout: split
---

<template v-slot:left>

```ts
type Success<T> = {
  success: true;
  value: T;
};

type Failure = {
  success: false;
  error: Error;
};

type Result<T> = Success<T> | Failure;

function ok<T>(value: T): Success<T> {
  return { success: true, value };
};

function err(error: Error): Failure {
  return { success: false, error };
};
```

</template>

<template v-slot:right>

```ts twoslash {monaco-write}
type Success<T> = {
  success: true;
  value: T;
};

type Failure = {
  success: false;
  error: Error;
};

type Result<T> = Success<T> | Failure;

function ok<T>(value: T): Success<T> {
  return { success: true, value };
};

function err(error: Error): Failure {
  return { success: false, error };
};

class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = "ValidationError";
  }
}

const alreadyChosenNames = ["Clem", "Clement", "Clément"]
// ---cut---
function validateName(name: string): Result<string> {
  if (alreadyChosenNames.includes(name)) {
    return err(new ValidationError(`Not available`));
  }
  return ok(name);
};

const result = validateName("Florian");

console.log(result.value)
```

</template>

---
layout: split
---

<template v-slot:left>

```ts
const toResult = (fn) => {
  return (...args) => {
    try {
      return ok(fn(...args));
    } catch (e) {
      return err(e);
    }
  };
}
```

</template>


<template v-slot:right>

```ts twoslash
type Success<T> = {
  success: true;
  value: T;
};

type Failure = {
  success: false;
  error: Error;
};

type Result<T> = Success<T> | Failure;

function ok<T>(value: T): Success<T> {
  return { success: true, value };
};

function err(error: Error): Failure {
  return { success: false, error };
};

const toResult = (fn) => {
  return (...args) => {
    try {
      return ok(fn(...args));
    } catch (e) {
      return err(e);
    }
  };
}
// ---cut---
const JSONParseResult = toResult(JSON.parse);

const parsingResult = JSONParseResult("`{}");

if (parsingResult.success) {
  console.log(parsingResult.value)
} else {
  console.error(parsingResult.error)
}
```

</template>

---
layout: split
---

<template v-slot:left>

## Typé

<br />

## Erreurs collectables

<br />

## Pas d'interruption du flux (verbeux), ni remontée d'erreurs

<div v-click="1">

<br />

<hr />

<br />

## Cas d'usage

<br />

### Erreurs nécessitant une action (Known / Stop)

<br />

### Code à usage interne

<br />

### Typescript obligatoire

</div>

</template>

<template v-slot:right>
<img src="/images/never-throw.png" alt="never throw" />

[https://github.com/supermacro/neverthrow](https://github.com/supermacro/neverthrow)
</template>

---
layout: statement
---

# FP

<!--
- Immutabilité

- Fonctions pures

- Pas de side-effects

- Composition de fonctions
-->

---
layout: statement
---

## Folktale

<br />

<img src="/images/folktale.png" style="height: 40vh" />

---
layout: section
---

## fp-ts

<br />

## Ramda

<br />

## Sanctuary

---
layout: split
---

## Result (Either)

<br />

<template v-slot:left>

```ts
import Result from "folktale/result/index.js";

const parseJSON = (input) => {
  try {
    return Result.Ok(JSON.parse(input));
  } catch (error) {
    return Result.Error(error.message);
  }
};

const extractName = (input) => {
  // => Result.Ok or Result.Error
}

const charsToDigits = (input) => {
  // => Result.Ok or Result.Error
};
```

</template>

<template v-slot:right>

```ts
const jsonToParse = `{"name": "Clem", "age": "34"}`;

const nameInDigits = 
  parseJSON(jsonToParse)
  .chain(extractName)
  .chain(charsToDigits)
  .matchWith({
    Ok: ({ value }) => value,
    Error: ({ value }) => {
      console.error(value);
    },
  });

console.log(nameInDigits);
```
</template>

---
layout: section
---

## Validation

```ts
import Validation from 'folktale/validation/index.js'
const { Success, Failure } = Validation;

const isPasswordLongEnough = (password) =>
  password.length > 6
    ? Success(password)
    : Failure([new Error("Password must have more than 6 characters.")]);

const isPasswordStrongEnough = (password) =>
  /[\W]/.test(password)
    ? Success(password)
    : Failure([new Error("Password must contain a special character.")]);

const isPasswordValid = (password) =>
  Success()
    .concat(isPasswordLongEnough(password))
    .concat(isPasswordStrongEnough(password));

console.log(isPasswordValid("foo").value);
// ==> [Error, Error]
```

---
layout: split
---

<template v-slot:left>

## Interruption du flux

<br />

## Erreurs collectables

<br />

## Gestion locale des erreurs

<br />

## Librairies externes et courbe d'apprentissage

<div v-click="1">

<br />

<hr />

## Cas d'usage

<br />

### Erreurs nécessitant une action (Known / Stop)

<br />

### Erreurs critiques (Unknown / Stop)

<br />

### Choix d'équipe et documentation des pratiques

</div>

</template>

<template v-slot:right>

<img src="/images/lambda.webp" alt="Lambda" style="" />

</template>

---
layout: statement
---

---
layout: statement
---

# Give some 💖 to your errors, they will pay you back
