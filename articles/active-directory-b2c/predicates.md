---
title: Predicados e PredicateValidations
titleSuffix: Azure AD B2C
description: Evite que dados malformados sejam adicionados ao seu inquilino Azure AD B2C usando políticas personalizadas no Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 887c9432f04cce775e045bb6da83f0af4a4a4bce
ms.sourcegitcommit: 632e7ed5449f85ca502ad216be8ec5dd7cd093cb
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/30/2020
ms.locfileid: "80396885"
---
# <a name="predicates-and-predicatevalidations"></a>Predicados e PredicateValidations

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Os **elementos Predicados** e **PredicatesValids** permitem que você execute um processo de validação para garantir que apenas dados devidamente formados sejam inseridos no inquilino B2C do Azure Active Directory (Azure AD B2C).

O diagrama a seguir mostra a relação entre os elementos:

![Diagrama mostrando relação Predicados e Validações predicadas](./media/predicates/predicates.png)

## <a name="predicates"></a>Predicados

O elemento **Predicado** define uma validação básica para verificar o valor de um tipo de declaração e retorna `true` ou `false`. A validação é feita usando um elemento **Método** especificado e um conjunto de elementos **Parâmetro** relevantes para o método. Por exemplo, um predicado pode verificar se o comprimento do valor da declaração de uma cadeia de caracteres está dentro do intervalo dos parâmetros mínimos e máximos especificados, ou se um valor de declaração de cadeia de caracteres contém um conjunto de caracteres. O elemento **UserHelpText** enviará uma mensagem de erro para os usuários se a verificação falhar. O valor do elemento **UserHelpText** pode ser localizado usando a [personalização de idioma](localization.md).

O elemento **Predicados** deve aparecer seguindo diretamente o elemento **ClaimsSchema** dentro do elemento [BuildingBlocks.](buildingblocks.md)

O elemento **Predicados** contém o seguinte elemento:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| Predicado | 1:n | Uma lista de predicados. |

O elemento **Predicado** contém os seguintes atributos:

| Atributo | Obrigatório | Descrição |
| --------- | -------- | ----------- |
| ID | Sim | Um identificador usado para o predicado. Outros elementos podem usar esse identificador na política. |
| Método | Sim | O tipo de método a ser usado para validação. Os valores possíveis: [IsLengthRange](#islengthrange), [MatchesRegex](#matchesregex), [IncludesCharacters](#includescharacters) ou [IsDateRange](#isdaterange).  |
| HelpText | Não | Uma mensagem de erro para os usuários se a verificação falhar. Essa cadeia de caracteres pode ser localizada usando a [personalização de idioma](localization.md) |

O elemento **Predicado** contém os seguintes elementos:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| UserHelpText | 0:1 | (Preterido) Uma mensagem de erro para os usuários se a verificação falhar. |
| parâmetros | 1:1 | Os parâmetros para o tipo de método de validação de cadeia de caracteres. |

O elemento **Parâmetros** contém os seguintes elementos:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| Parâmetro | 1:n | Os parâmetros para o tipo de método de validação de cadeia de caracteres. |

O elemento **Parâmetro** contém os seguintes atributos:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| ID | 1:1 | O identificador do parâmetro. |

### <a name="predicate-methods"></a>Métodos predicados

#### <a name="islengthrange"></a>IsLengthRange

O método IsLengthRange verifica se o comprimento de um valor de reclamação de string está dentro do intervalo de parâmetros mínimos e máximos especificados. O elemento predicado suporta os seguintes parâmetros:

| Parâmetro | Obrigatório | Descrição |
| ------- | ----------- | ----------- |
| Máximo | Sim | O número máximo de caracteres que podem ser inseridos. |
| Mínimo | Sim | O número mínimo de caracteres que devem ser inseridos. |


O exemplo a seguir mostra um `Minimum` método `Maximum` IsLengthRange com os parâmetros e que especifica o intervalo de comprimento da string:

```XML
<Predicate Id="IsLengthBetween8And64" Method="IsLengthRange" HelpText="The password must be between 8 and 64 characters.">
  <Parameters>
    <Parameter Id="Minimum">8</Parameter>
    <Parameter Id="Maximum">64</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="matchesregex"></a>JogosRegex

O método MatchesRegex verifica se um valor de reclamação de string corresponde a uma expressão regular. O elemento predicado suporta os seguintes parâmetros:

| Parâmetro | Obrigatório | Descrição |
| ------- | ----------- | ----------- |
| RegularExpression | Sim | O padrão de expressão regular para correspondência. |

O exemplo a seguir mostra um método `MatchesRegex` com o parâmetro `RegularExpression` que especifica uma expressão regular:

```XML
<Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be numbers only.">
  <Parameters>
    <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="includescharacters"></a>Inclui personagens

O método IncludesCharacters verifica se um valor de reclamação de seqüência contém um conjunto de caracteres. O elemento predicado suporta os seguintes parâmetros:

| Parâmetro | Obrigatório | Descrição |
| ------- | ----------- | ----------- |
| CharacterSet | Sim | O conjunto de caracteres que podem ser inseridos. Por exemplo, caracteres `a-z`minúsculos, caracteres maiúsculos, `A-Z`dígitos `0-9`ou uma lista de símbolos, tais como `@#$%^&amp;*\-_+=[]{}|\\:',?/~"();!`. |

O exemplo a seguir mostra um método `IncludesCharacters` com o parâmetro `CharacterSet` que especifica o conjunto de caracteres:

```XML
<Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
  <Parameters>
    <Parameter Id="CharacterSet">a-z</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="isdaterange"></a>IsDateRange

O método IsDateRange verifica se um valor de solicitação de data está entre uma faixa de parâmetros mínimos e máximos especificados. O elemento predicado suporta os seguintes parâmetros:

| Parâmetro | Obrigatório | Descrição |
| ------- | ----------- | ----------- |
| Máximo | Sim | A maior data possível que pode ser inserida. O formato da data `yyyy-mm-dd` segue `Today`a convenção, ou . |
| Mínimo | Sim | A menor data possível que pode ser inserida. O formato da data `yyyy-mm-dd` segue `Today`a convenção, ou .|

O exemplo a seguir mostra um método `IsDateRange` com os parâmetros `Minimum` e `Maximum` que especificam o intervalo de datas com um formato de `yyyy-mm-dd` e `Today`.

```XML
<Predicate Id="DateRange" Method="IsDateRange" HelpText="The date must be between 1970-01-01 and today.">
  <Parameters>
    <Parameter Id="Minimum">1970-01-01</Parameter>
    <Parameter Id="Maximum">Today</Parameter>
  </Parameters>
</Predicate>
```

## <a name="predicatevalidations"></a>PredicateValidations

Embora os predicados definam a validação para verificar um tipo de declaração, **PredicateValidations** agrupa um conjunto de predicados para formar uma validação de entrada do usuário que pode ser aplicada a um tipo de declaração. Cada elemento **PredicateValidation** contém um conjunto de elementos **PredicateGroup** que contém um conjunto de elementos **PredicateReference** que apontam para um **Predicado**. Para passar a validação, o valor da declaração deve passar em todos os testes de qualquer predicado em todos os **PredicateGroup** com seu conjunto de elementos **PredicateReference**.

O **elemento PredicateValidations** deve aparecer seguindo diretamente o elemento **Predicados** dentro do elemento [BuildingBlocks.](buildingblocks.md)

```XML
<PredicateValidations>
  <PredicateValidation Id="">
    <PredicateGroups>
      <PredicateGroup Id="">
        <UserHelpText></UserHelpText>
        <PredicateReferences MatchAtLeast="">
          <PredicateReference Id="" />
          ...
        </PredicateReferences>
      </PredicateGroup>
      ...
    </PredicateGroups>
  </PredicateValidation>
...
</PredicateValidations>
```

O elemento **PredicateValidations** contém o seguinte elemento:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| PredicateValidation | 1:n | Uma lista de validação de predicado. |

O elemento **PredicateValidation** contém o seguinte atributo:

| Atributo | Obrigatório | Descrição |
| --------- | -------- | ----------- |
| ID | Sim | Um identificador usado para a validação do predicado. O elemento **ClaimType** pode usar esse identificador na política. |

O elemento **PredicateValidation** contém o seguinte elemento:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| PredicateGroups | 1:n | Uma lista de grupos de predicado. |

O elemento **PredicateGroups** contém o seguinte elemento:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| PredicateGroup | 1:n | Uma lista de predicados. |

O elemento **PredicateGroup** contém o seguinte atributo:

| Atributo | Obrigatório | Descrição |
| --------- | -------- | ----------- |
| ID | Sim | Um identificador usado para o grupo de predicados.  |

O elemento **PredicateGroup** contém os seguintes elementos:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| UserHelpText | 0:1 |  Uma descrição do predicado que pode ser útil para os usuários saberem o valor que devem digitar. |
| PredicateReferences | 1:n | Uma lista de referências de predicado. |

O elemento **PredicateReferences** contém os seguintes atributos:

| Atributo | Obrigatório | Descrição |
| --------- | -------- | ----------- |
| MatchAtLeast | Não | Especifica que o valor deve corresponder a pelo menos à quantidade de definições de predicado para a entrada ser aceita. Se não for especificado, o valor deve corresponder a todas as definições predicadas. |

O elemento **PredicateReferences** contém os seguintes elementos:

| Elemento | Ocorrências | Descrição |
| ------- | ----------- | ----------- |
| PredicateReference | 1:n | Uma referência a um predicado. |

O elemento **PredicateReference** contém os seguintes atributos:

| Atributo | Obrigatório | Descrição |
| --------- | -------- | ----------- |
| ID | Sim | Um identificador usado para a validação do predicado.  |


## <a name="configure-password-complexity"></a>Configurar a complexidade de senha

Com **Predicados** e **PredicateValidationsInput**, é possível controlar os requisitos de complexidade para senhas fornecidas por um usuário ao criar uma conta. Por padrão, o Azure AD B2C usa senhas fortes. O Azure AD B2C também oferece suporte a opções de configuração para controlar a complexidade de senhas que os clientes podem usar. É possível definir a complexidade de uma senha usando estes elementos predicados:

- **IsLengthBetween8And64** usando o método `IsLengthRange`, valida que senha precisa ter entre 8 e 64 caracteres.
- **Lowercase** usando o método `IncludesCharacters`, valida que a senha contém uma letra minúscula.
- **Uppercase** usando o método `IncludesCharacters`, valida que a senha contém uma letra maiúscula.
- **Number** usando o método `IncludesCharacters`, valida que a senha contém um dígito.
- **Símbolo** usando `IncludesCharacters` o método, valida que a senha contém um dos vários caracteres símbolo.
- **PIN** usando o método `MatchesRegex`, valida que a senha contém somente números.
- **AllowedAADCharacters** usando o método `MatchesRegex`, valida que o único caractere inválido da senha foi fornecido.
- **DisallowedWhitespace** usando o método `MatchesRegex`, valida que a senha não começa ou termina com um caractere de espaço em branco.

```XML
<Predicates>
  <Predicate Id="IsLengthBetween8And64" Method="IsLengthRange" HelpText="The password must be between 8 and 64 characters.">
    <Parameters>
      <Parameter Id="Minimum">8</Parameter>
      <Parameter Id="Maximum">64</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">a-z</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Uppercase" Method="IncludesCharacters" HelpText="an uppercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">A-Z</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Number" Method="IncludesCharacters" HelpText="a digit">
    <Parameters>
      <Parameter Id="CharacterSet">0-9</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Symbol" Method="IncludesCharacters" HelpText="a symbol">
    <Parameters>
      <Parameter Id="CharacterSet">@#$%^&amp;*\-_+=[]{}|\\:',.?/`~"();!</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be numbers only.">
    <Parameters>
      <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="AllowedAADCharacters" Method="MatchesRegex" HelpText="An invalid character was provided.">
    <Parameters>
      <Parameter Id="RegularExpression">(^([0-9A-Za-z\d@#$%^&amp;*\-_+=[\]{}|\\:',?/`~"();! ]|(\.(?!@)))+$)|(^$)</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="DisallowedWhitespace" Method="MatchesRegex" HelpText="The password must not begin or end with a whitespace character.">
    <Parameters>
      <Parameter Id="RegularExpression">(^\S.*\S$)|(^\S+$)|(^$)</Parameter>
    </Parameters>
  </Predicate>
```

Após definir as validações básicas, você poderá combiná-las e criar um conjunto de políticas de senha a serem usadas na sua política:

- **SimplePassword** valida DisallowedWhitespace, AllowedAADCharacters e IsLengthBetween8And64
- **StrongPassword** valida DisallowedWhitespace, AllowedAADCharacters, IsLengthBetween8And64. O último grupo `CharacterClasses` executa um conjunto adicional de predicados com `MatchAtLeast` definido como 3. A senha do usuário precisa ter entre 8 e 16 caracteres e três dos seguintes caracteres: minúsculas, maiúsculas, números ou símbolos.
- **CustomPassword** valida somente DisallowedWhitespace, AllowedAADCharacters. Portanto, o usuário pode fornecer qualquer senha com qualquer comprimento, desde que os caracteres sejam válidos.

```XML
<PredicateValidations>
  <PredicateValidation Id="SimplePassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences>
          <PredicateReference Id="IsLengthBetween8And64" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>

  <PredicateValidation Id="StrongPassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
       </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences>
          <PredicateReference Id="IsLengthBetween8And64" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="CharacterClasses">
        <UserHelpText>The password must have at least 3 of the following:</UserHelpText>
        <PredicateReferences MatchAtLeast="3">
          <PredicateReference Id="Lowercase" />
          <PredicateReference Id="Uppercase" />
          <PredicateReference Id="Number" />
          <PredicateReference Id="Symbol" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>

  <PredicateValidation Id="CustomPassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

No seu tipo de declaração, adicione o elemento **PredicateValidationReference** e especifique o identificador como uma das validações de predicado, como SimplePassword, StrongPassword ou CustomPassword.

```XML
<ClaimType Id="password">
  <DisplayName>Password</DisplayName>
  <DataType>string</DataType>
  <AdminHelpText>Enter password</AdminHelpText>
  <UserHelpText>Enter password</UserHelpText>
  <UserInputType>Password</UserInputType>
  <PredicateValidationReference Id="StrongPassword" />
</ClaimType>
```

O exemplo a seguir mostra como os elementos são organizados quando o Azure AD B2C exibe a mensagem de erro:

![Diagrama de exemplo de complexidade de senha predicado e predicado](./media/predicates/predicates-pass.png)

## <a name="configure-a-date-range"></a>Configurar um intervalo de datas

Com os elementos **Predicates** e **PredicateValidations**, é possível controlar os valores de data mínimos e máximos do **UserInputType** usando um `DateTimeDropdown`. Para fazer isso, crie um **Predicado** com o `IsDateRange` método e forneça os parâmetros mínimos e máximos.

```XML
<Predicates>
  <Predicate Id="DateRange" Method="IsDateRange" HelpText="The date must be between 01-01-1980 and today.">
    <Parameters>
      <Parameter Id="Minimum">1980-01-01</Parameter>
      <Parameter Id="Maximum">Today</Parameter>
    </Parameters>
  </Predicate>
</Predicates>
```

Adicione um **PredicateValidation** com uma referência ao predicado `DateRange`.

```XML
<PredicateValidations>
  <PredicateValidation Id="CustomDateRange">
    <PredicateGroups>
      <PredicateGroup Id="DateRangeGroup">
        <PredicateReferences>
          <PredicateReference Id="DateRange" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

No seu tipo de declaração, adicione o elemento **PredicateValidationReference** e especifique o identificador como `CustomDateRange`.

```XML
<ClaimType Id="dateOfBirth">
  <DisplayName>Date of Birth</DisplayName>
  <DataType>date</DataType>
  <AdminHelpText>The user's date of birth.</AdminHelpText>
  <UserHelpText>Your date of birth.</UserHelpText>
  <UserInputType>DateTimeDropdown</UserInputType>
  <PredicateValidationReference Id="CustomDateRange" />
</ClaimType>
 ```

## <a name="next-steps"></a>Próximas etapas

- Saiba como configurar a [complexidade da senha usando políticas personalizadas no Azure Active Directory B2C](custom-policy-password-complexity.md) usando validações de predicados.