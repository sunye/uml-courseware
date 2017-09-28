# OCL
The Object Constraint Language

note:
    Speaker notes.

[//]: # (TODO)
[//]: #  (Tuples)
[//]: #  (Messages)
[//]: #  (Vérification si un attribut est nul)

----

## Plan
  1. **Introduction**
  2. Principles
  2. Invariants
  3. Property Definition
  4. Operation Specification
  5. Advanced Topics
  6. Conclusion
  7. Appendix - Language Details

----

## Introduction

----

## What is OCL?

OCL stands for «Object Constraint Language». It is:
- a OMG standard (see http://www.omg.org/spec/OCL/).
- a formal and unambiguous language, but easy to use (even for non mathematicians).
- a complement to UML (and also to MOF, but that is another history).

----
## Why do I need OCL?
Sometimes, the UML lacks precision. Suppose the following class diagram:

![](../resources/png/family.png)

- How do you specify that this class only considers people born after 1900?
- And how do you specify that cycles are not allowed (i.e., that a person cannot be an ancestral of himself)?

----
## What About Comments?

- Comments, expressed in natural languages, are often very useful.
- But sometimes, they are also ambiguous:
![](../resources/png/marriage.png)

- Still, comments cannot avoid some situations:

![](../resources/png/anna-bob-carol.png)

----

## How can OCL add more precision to UML?

- By adding *constraints* to modeling elements:

```ocl
context Person
inv: self.wife->notEmpty() implies self.wife.husband = self and
    self.husband->notEmpty() implies self.husband.wife = self
```

![](../resources/png/marriage.png)

----
## Some Basic Principles
- OCL expressions have no side effect



## Plan
  1. Introduction
  2. Invariants
  3. Model
  3. Property Definition
  4. Operation Specification
  5. Advanced Topics
  6. Conclusion
  7. Appendix - Language Details

----

## Basic Principles

----
## Collections and Elements

- A OCL expression refers to the following constituents:
  - Values of basic types: `Integer`, `Real`, `Boolean`, `String`, `UnlimitedNatural`;
  - Modeling elements, from the associated UML model;
  - Collections of values or modeling elements.
- Operation calls on elements and values use dots:
```ocl
'Nantes'.substring(1,3) = 'Nan'
```
- Operation calls on collections use arrows:
```ocl
{1, 2, 3, 4, 5}->size() = 5
```

----


----
## Exemple de modèle

![](../resources/png/universite.png)


----


## Plan
  1. Introduction
  2. Basic Principles
  2. **Invariants**
  3. Model Navigation
  3. Property Definition
  4. Operation Specification
  5. Advanced Topics
  6. Conclusion
  7. Appendix - Language Details

----

## Invariants

----
## Class Invariants

- A class invariant is a constraint that must be verified by all instances of a class, when in a **stable state**.

- The notion of stable state is important: an invariant may be broken during the execution of an operation.

- It is commonly accepted that an instance is in a stable state between the execution of two public operations.

----

## Invariants: Graphical Notation

Invariants can be placed directly on the modeling element, between braces ({}) or on a comment attached to it:

![](../resources/png/person-inv.png)

![](../resources/png/person-inv-note.png)

----
## Invariants: Textual Notation

Invariants may also be placed on a separate document. In this case, the notion of **context** is important.

```ocl
context Person inv: self.age < 150
context Person inv: age < 150
```

----
## Notion of Context

- Every OCL expression is attached to a specific **context**: a UML modeling element.
- The context may be referenced inside the expression using the `self` keyword.

```ocl
context Person inv: self.age < 150
context Person inv: self.name.size() > 0
```

----

##  Context Properties

- The context allows the access to some properties from the attached modeling element.
- In the case of a UML class, this means: attributes, query operations, and states (from attached state machines).


![](../resources/png/person.png)

```ocl
context Person
inv:
  self.name.size() > 1 and
  self.age() >= 0 and
  self.oclInState(Single)
```
note:
  While it is possible to check a state within an invariant, this is not logical.

----

## Invariants de classe

[comment]:# (%\item Expression OCL stéréotypée \og invariant \fg.)

- Exemples:

```ocl
context e : Etudiant inv: ageMinimum: e.age > 16
context e : Etudiant inv: e.age > 16
context Etudiant inv: self.age > 16
context Etudiant inv: age > 16
```

----
## Plan
  1. Introduction
  2. Principles
  2. Invariants
  3. **Model Navigation**
  3. Property Definition
  4. Operation Specification
  5. Advanced Topics
  6. Conclusion
  7. Appendix - Language Details

----

## Model Navigation

----
## Role Navigation

Il est possible de naviguer à travers les associations, en utilisant le rôle opposé:

![](../resources/png/univ-departement.png)

```ocl
 context Departement
 	inv: self.universite->notEmpty()

 context Universite
 	inv: self.departement-> (...)
```

----

## Cardinalities

Le type de la valeur de l'expression dépend de la cardinalité maximale du rôle.
Si égal à 1, alors c'est un classificateur. Si > 1, alors c'est une collection.


![](../resources/png/matiere.png)

```ocl
context Matiere
	-- un objet:
	inv: self.enseignant.oclInState(disponible)

	-- une collection (Set):
	inv: self.est_maitrisee->notEmpty()
```

----

## Role names

- Quand le nom de rôle est absent, le nom du type (en minuscule) est utilisé.
- Il est possible de naviguer sur des rôles de  cardinalité 0 ou 1 en tant que collection:

```ocl
context Departement inv: self.chef->size() = 1

context Departement inv: self.chef.age > 40

context Personne inv: self.epouse->notEmpty()
    implies self.epouse.sexe = Sexe::femme
```

----

## Rôles: navigation

- Il est possible de combiner des expressions:


```ocl
context Personne inv:
    self.epouse->notEmpty() implies self.epouse.age >= 18 and

    self.mari->notEmpty() implies self.mari.age >= 18
```

----

## Association-Classes

- On utilise le nom de la classe-association, en minuscules:

```ocl
context Etudiant
inv:
    -- La moyenne des notes d'un étudiant est toujours supérieure à 4:
    self.note.valeur->average() > 4
```
![](../resources/png/note.png)


----

## Association-Classes

- Il est possible de naviguer à partir de la classe-association en utilisant les noms de rôle et la notation pointée:

```ocl
context Note inv:
    self.etudiant.age() >= 18
    self.matiere.heures > 3
```
![](../resources/png/note.png)


----

## Qualified Associations

- La valeur du qualificatif est mise entre crochets:
```ocl
context Universite
    -- Le nom de l'étudiant 8764423 est "Martin".
    inv: self.etudiants[8764423].nom = "Martin"
```
- Quand la valeur n'est pas précisée, le résultat est une collection:
```ocl
context Universite
    -- Il existe un étudiant dont le nom est "Martin"
    inv: self.etudiants->exists(each | each.nom = "Martin")
```

![](../resources/png/asso-qualifiee.png)


----
## Plan
  1. Introduction
  2. Principles
  2. Invariants
  3. **Property Definition**
  4. Operation Specification
  5. Advanced Topics
  6. Conclusion
  7. Appendix - Language Details

----

## Property Definition, Intialization, and Calculation

----

## Property Definition

- OCL allows the definition of new attributes and new operations, and add them to an existing class.
- These new properties can be used within other OCL constraints.

Syntax:
```ocl
context <class-name>
  def: <attr-name> : <type> = <ocl-expression>
  def: <op-name> (<argument-list) : type = <ocl-expression>
```

----

## Property Definition

- Useful to decompose complex expressions without overloading the model.

Examples:
```ocl
context Enseignant
def: eleves() : Bag(Etudiants) = self.enseigne.etudiant

context Departement
def: eleves():Set(Etudiants) = self.enseignants.enseigne.etudiant->asSet()
```

<!--
inv: self.titre = Titre::prof implies self.eleves()
    ->forAll(each | each.estAdmis())
    -- un professeur a toujours 100\% de reussite
-->

----

## Property Initialization

- Initial value specification for attributes  and roles (association ends).

- The expression type must conform to the attribute or role type.

Syntax:
```ocl
context <class-name>::<prop-name>: <type>
    init: <ocl-expression>
```
Example:
```
context Enseignant::salaire : Integer
    init: 800
```

----

## Derived Property Specification

- OCL expression defining how a derived property is calculated.

Syntax:
```ocl
context <class-name>::<role-name>: <type>
    derive:  <ocl-expression>
```

Examples:
```ocl
context Enseignant::service : Integer
    derive: self.enseigne.heures->sum()

context Personne::celibataire : Boolean
    derive: self.conjoint->isEmpty()
```

----

## Query Operation Specification

- Specification of query operation body.

Example:
```
context Universite::enseignants() : Set(Enseignant)
body:
    self.departements.enseignants->asSet()
```

----

## Plan
  1. Introduction
  2. Principles
  2. Invariants
  3. Property Definition
  4. **Operation Specification**
  5. Advanced Topics
  6. Conclusion
  7. Appendix - Language Details

----

## Operation Specification

----

## Operation Specification

- Inspirée des types abstraits: une opération est composée d'une signature, de pré-conditions et de post-conditions.
- Permet de contraindre l'ensemble de valeurs d'entrée d'une opération.
- Permet de spécifier la sémantique d'une opération: ce qu'elle fait et non comment elle le fait.


----

## Preconditions

- Pré-condition d'une opération (ou d'une transition).
  - Une contrainte qui doit être toujours vraie
**avant** l'exécution d'une opération.
- Ce qui doit être respecté par le client (l'appelant de l'opération)
- Représentée par une expression OCL stéréotypée «precondition».

```ocl
context Departement::ajouter(e : Enseignant) : Integer
    pre nonNul: not e.oclIsUndefined()
```

----

## Postconditions

- Post-condition d'une opération (ou d'une transition).
  - Une contrainte qui doit être toujours vraie **après** l'exécution d'une opération.
- Spécifie ce qui devra être vérifié après l'exécution d'une opération.
- Représentée par une expression OCL stéréotypée «postcondition»:
- Opérateurs spéciaux:
  - `@pre`: accès à une valeur d'une propriété d'avant l'opération (old de Eiffel).
  - `result`: accès au résultat de l'opération.

----

## Postcondition


```ocl
context Etudiant::age() : Integer
post correct: result = (today - naissance).years()

context Typename::operationName(param1: type1, ...): Type
post: result = ...

context Typename::operationName(param1: type1, ...): Type
post resultOk: result = ...
```

----

## Previous Values

A l'intérieur d'une postcondition, deux valeurs sont disponibles pour chaque propriété:

- la valeur de la propriété avant l'opération.
- la valeur de la propriété après la fin de l'opération.

```ocl
context Personne::anniversaire()
	post: age = age@pre + 1

context Enseignant::augmentation(v : Integer)
	post: self.salaire = self.salaire@pre + v
```

----



## Previous Values (1/2)

Quand la valeur `@pre` d'une propriété est un objet, toutes les valeurs
atteintes à partir de cet objet sont nouvelles:

```ocl
a.b@pre.c
		-- l'ancienne valeur de b, disons X,
		-- et la nouvelle valeur de c de X

a.b@pre.c@pre   
		-- l'ancienne valeur de b, disons X,
		-- et l'ancienne valeur de c de x.
```

----
## Previous Values (2/2)

![](../resources/png/atpre.png)

```ocl
a.b@pre.c -- la nouvelle valeur de b1.c,  c3		
a.b@pre.c@pre  -- l'ancienne valeur de b1.c, c1
a.b.c -- la nouvelle valeur de b2.c, c2
```

----

## Plan
  1. Introduction
  2. Principles
  2. Invariants
  3. Property Definition
  4. Operation Specification
  5. **Advanced Topics**
  6. Conclusion
  7. Appendix - Language Details

----

## Advanced Topics

Tuples, Messages, Constraint Inheritance

----

## Tuples

Définition

**Tuple:**

    Une N-Uplet est une séquence finie de objets ou *composantes*, où chaque composante est nommée.
    Les types des composantes sont potentiellement différents.


**Exemples:**

```ocl
Tuple {nom:String = 'Martin', age:Integer = 42}
Tuple {nom:'Colette', notes:Collection(Integer) = Set{12, 13, 9},
     formation:String = 'Informatique'}
```

----

## N-Uplets

Notation

Les types sont optionnels. L'ordre des composantes n'est pas important:

**Expressions équivalentes:**
```
Tuple {name: String = 'Martin,' age: Integer = 42}
Tuple {name = 'Martin,' age = 42}
Tuple {age = 42, name = 'Martin'}
```

----

## N-Uplets

Les valeurs des composantes peuvent être spécifiées par des expressions OCL:

```ocl
context Universite def:
statistiques : Set(Tuple(dpt:Departement, nbEtudiants:Integer,
                               admis: Set(Etudiants), moyenne: Integer)) =
     departement->collect(each |
       Tuple {dpt:Departement = each,
           nbEtudiants: Integer = each.eleves()->size(),
           admis: Set(Person) = each.eleves()->select(estAdmis()),
           moyenne: Integer = eleves().note->avg()
          }
      )
```

----

## N-Uplets

**Notation**

Les composantes sont accessibles grâce à leurs noms, en utilisant la notation pointée:

```ocl
Tuple {nom:String='Martin', age:Integer = 42}.age = 42
```

L'attribut `statistiques` définit précédemment peut être utilisé à l'intérieur d'un autre expression OCL:

```ocl
context Universite inv:
     statistiques->sortedBy(moyenne)->last().dpt.nom = 'Informatique'
     -- Le département d'informatique possède les meilleurs étudiants.
```

----

<!--
%\section{Messages}
%\frame{\tableofcontents[currentsection,hideothersubsections]}

%        \item  Envoi de messages.
%    \end{itemize}
%\begin{ocl}
%context Subject::hasChanged()
%	-- la post-condition doit assurer que le message update()
%	-- a ete envoye a tous les observateurs:
%	post: observers->forAll( o | o^update() )
%\end{ocl}

% \begin{frame}
% \frametitle{Messages}
% %\framesubtitle{}
% A partir de la version 2.0, OCL introduit la notion de \emph{message}, qui permet de mieux lier la partie structurelle d'un modèle à son comportement.
% \end{frame}
-->

## Messages

Appel d'opérations et envoi d'événements

 On utilise l'opérateur `^` (hasSent) pour spécifier qu'une communication a eu lieu.

```ocl
context Subject::hasChanged()
post:  observer^update(12, 14)
```

<!--
% L'expression  \code{observer^update(12, 14)} est évaluée vraie si le message \code{update()} avec les arguments 12 et 14 a été envoyée à l'objet «observer». L'expression \code{update()} est soit une opération définie dans la classe de l'objet «observer», soit un signal spécifié dans le modèle UML.

% Les arguments du message (12 et 14) doivent être conformes aux paramètres de l'opération ou du signal.
-->

----
## Messages

Jokers

[comment]: # (% Si les arguments ne sont pas connus, ou ne sont pas restreints, ils peuvent ne pas être spécifiés. Dans ce cas, in utilise un point d'interrogation.)

[comment]: # (%Following the question mark is an optional type, which may be needed to find the  correct operation when the same operation exists with different parameter types.)

```ocl
context Subject::hasChanged()
post:  observer^update(? : Integer, ? : Integer)
```

L'opérateur «?» indique que les valeurs des arguments ne sont pas connues.

----
## Le type OclMessage
OCL introduit le type `OclMessage`.
L'opérateur \messages permet d'accéder à une séquence de messages envoyés.

[comment]: # (%OCL also defines a special OclMessage type. One can get the actual OclMessages through the message operator:  \messages.)

```ocl
observer^^update(?, ?)
-- renvoie une séquence de messages envoyés.
```

[comment]: # (% This results in the Sequence of messages sent. Each element of the collection is an instance of OclMessage. In the remainder of the constraint one can refer to the parameters of the operation using their formal parameter name from the operation definition.)


----
## Messages
Valeurs des arguments

Il est possible d'accéder aux valeurs des arguments d'un message grâce aux noms des paramètres de l'opération ou du signal:

```ocl
context Subject::hasChanged()
post: let messages : Sequence(OclMessage) =
            observer^^update(? : Integer, ? : Integer) in
      messages->notEmpty() and
      messages->exists( m | m.i > 0 and m.j >= m.i )
```

<!--
%The value of the parameter i is not known, but it must be greater than zero and the value of parameter j must be larger or equal to i.

%\begin{frame}[fragile]
%\frametitle{Messages}

%Because the \messages operator results in an instance of OclMessage, the message expression can also be used to specify
%collections of messages sent to different targets. For an observer pattern we can write:

%\begin{ocl}
%context Subject::hasChanged()
%post:  let messages : Sequence(OclMessage) =
%                    observers->collect(o | o^^update(? : Integer, ? : Integer) ) in
%       messages->forAll(m | m.i <= m.j )
%\end{ocl}
%
%Messages is now a set of OclMessage instances, where every OclMessage instance has one of the observers as a target.
%\end{frame}
-->

----
## Messages

Accès aux valeurs renvoyés

L'opérateur `OclMessage::result()` permet l'accès à la valeur renvoyée d'une opération (les signaux ne renvoient pas de valeur).
L'opérateur `OclMessage::hasReturned()` retourne vrai si l'opération a renvoyé une valeur.

<!--
% A signal sent message is by definition asynchronous, so there never is a return value. If there is a logical return value it  must be modeled as a separate signal message. Yet, for an operation call there is a potential return value. This is only available if the operation has already returned (not necessary if the operation call is asynchronous), and it specifies a return type in its definition. The standard operation result() of OclMessage contains the return value of the called operation. If \code{getMoney(...)} is an operation on Company that returns a boolean, as in \code{Company::getMoney(amount : Integer) : Boolean}, we can write:
-->

```ocl
context Person::giveSalary(amount : Integer)
post: let message : OclMessage = company^getMoney(amount) in
      message.hasReturned()
      -- getMoney was sent and returned
      and
      message.result()
      -- the getMoney call returned true
```

<!--
%As with the previous example we can also access a collection of return values from a collection of OclMessages. If message.hasReturned() is false, then message.result() will be undefined.
\end{frame}
-->

<!--
%\begin{frame}[fragile]
%\frametitle{Messages}
%\framesubtitle{An example}
%This section shows an example of using the OCL message expression.
%The Example and Problem
%Suppose we have build a component, which takes any form of input and transforms it into garbage (aka encrypts it). The
%component GarbageCan uses an interface UsefulInformationProvider that must be implemented by users of the
%component to provide the input. The operation getNextPieceOfGarbage of GarbageCan can then be used to retrieve the
%garbled data. Figure 7.5 shows the component's class diagram. Note that none of the operations are marked as queries.

%When selling the component, we do not want to give the source code to our customers. However, we want to specify the
%component?s behavior as precisely as possible. So, for example, we want to specify, what getNextPieceOfGarbage does.
%Note that we cannot write:

%\begin{ocl}
%context GarbageCan::getNextPieceOfGarbage() : Integer
%post: result = (datasource.getNextPieceOfData() * .7683425 + 10000) / 20 + 3
%\end{ocl}

%because \code{UsefulInformationProvider::getNextPieceOfData()} is not a query (e.g., it may increase some internal pointer so
%that it can return the next piece of data at the next call). Still we would like to say something about how the garbage is
%derived from the original data.
%\end{frame}
-->

<!--
%\begin{frame}[fragile]
%\frametitle{Messages}
%\framesubtitle{An example}
%The solution
%To solve this problem, we can use an OclMessage to represent the call to getNextPieceOfData. This allows us to check for
%the result. Note that we need to demand that the call has returned before accessing the result:

%\begin{ocl}
%context GarbageCan::getNextPieceOfGarbage() : Integer
%post: let message : OclMessage = datasource^^getNextPieceOfData()->first() in
%      message.hasReturned()
%      and
%      result = (message.result() * .7683425 + 10000) / 20 + 3
%\end{ocl}

%\end{frame}
-->


<!--
## Stéréotypes des contraintes

Plusieurs stéréotypes sont définis en standard dans UML:

- Invariants de classe: «invariant»
- Pré-conditions: «precondition»
- Post-conditions: «postcondition»
- Définitions de propriétés: «definition»


----
## Package context

Il est possible de spécifier explicitement le nom du paquetage auquel appartient une contrainte:

```ocl
package Package::SubPackage
context X inv:
    -- some invariant

context X::operation()
pre:
    -- some precondition
endpackage
```

-->


----



## Constraint Inheritance

**Liskov substitution principle (LSP)**

> «Partout où une instance d'une classe est attendue, il est possible d'utiliser une instance d'une de ses sous-classes.»


----
## Invariant Inheritance

Conséquences du principe de substitution de Liskov sur les invariants:

- Les invariants sont toujours hérités par les sous-classes.

- Une sous-classe peut renforcer l'invariant.

----
## Pre- and Post-Condition Inheritance

Conséquences du LSP pour les pré et post-conditions:

- Une pré-condition peut seulement être assouplie (contrevariance).
- Une post-condition peut seulement être renforcée


----
## Plan

1. Introduction
2. Principles
2. Invariants
3. Property Definition
4. Operation Specification
5. Advanced Topics
5. **Conclusion**
6. Appendix - Language Details

----

## Conclusion

----

##  OCL Usages

OCL expression can specify:

- Class invariants;
- Class attribute initialization;
- Class derived attributes;
- New class properties: attributes and _query_ operations;
- Class operations pre- and post-conditions;
- Transition guards;
- Transition pre- and post-conditions;

note:
    A _query_ operation is operation with no side effect.

----


## Conseils de modélisation

- Faire simple: les contrats doivent améliorer la qualité des spécifications et non les rendre plus complexes.

- Toujours combiner OCL avec un langage naturel: les contrats servent à rendre les commentaires moins ambigus et non à les remplacer.

- Utiliser un outil.

----
## Rappels

La conception par contrats permet aux concepteurs de~:

-  Modéliser de manière plus précise;
-  Mieux documenter un modèle;
-  Rester indépendant de l'implémentation;
-  Identifier les responsabilités de chaque composant.

----

## Applicabilité

- Génération de code:
  - assertions en Eiffel, Sather.
  - dans d'autres langages, grâce à des outils spécialisés: iContract, JMSAssert, jContractor, Handshake, Jass, JML, JPP, etc.

- Génération de tests mieux ciblés.

----

## References

- The Object Constraint Language \--- Jos Warmer, Anneke Kleppe.
- OCL home page: http://www.klasse.nl/ocl/
- OCL tools: http://www.um.es/giisw/ocltools
- OMG Specification v2.3.1 http://www.omg.org/spec/OCL/Current/
- OMG UML 2.5 Working Group.   

----

## Tools

- Eclipse OCL. https://projects.eclipse.org/projects/modeling.mdt.ocl

- OCL Checker (Klasse Objecten)

- USE OCL (Mark Richters). http://useocl.sourceforge.net/w/

- Dresden OCL. http://www.dresden-ocl.org
- Octopus (Warmer & Kleppe). http://octopus.sourceforge.net/

----
## Plan

1. Introduction
2. Principles
2. Invariants
3. Property Definition
4. Operation Specification
5. Advanced Topics
5. Conclusion
6. **Appendix - Language Details**


## Appendix
Language Details

----
## Access to Class-level Properties
- Class-level properties are accessed through double-colons.

Class-level attributes:

```ocl
context Enseignant inv:
	self.salaire < Enseignant::salaireMaximum
```

Class-level query operations:
```ocl
context Etudiant inv:
	self.age() > Etudiant::ageMinimum()
```

----
## Access to Enumeration Literals and Nested States

- To avoid name conflicts, enumeration literals are preceded by the enumeration name and double-colons:

```ocl
context Enseignant inv:
	self.titre = Titre::prof implies
		self.salaire > 10
```

- Nested states (from the attached state machine) are preceded by the container state name and double-colons:

```ocl
context Departement::ajouter(e:Enseignant)
	pre:e.oclInState(Unavailable::Holydays)
	-- etats imbriques
```
----


## Basic Types

Type        | Values
--|--
`OclInvalid` | invalid
`OclVoid`   | null, invalid
`Boolean`  |  true, false
`Integer`  |  1, -5, 2, 34, 26524, etc.
`Real`  |  1.5, 3.14, etc.
`String`  |  "To be or not to be..."
`UnlimitedNatural`  |  0, 1, 2, 42, ... , *

----

## Collection Types (1/2)

Type | Description | Obtained from | Examples
--|--|--|--
Set  | unordered set.  | Simple navigation  |  \{1, 2, 45, 4\}
OrderedSet  |  ordered set. | Navigation through an ordered association end (labelled with `{ordered}`)  |  \{1, 2, 4, 45\}

----

## Collection Types (2/2)

Type | Description | Obtained from | Examples
--|--|--|--
Bag  | unordered multiset.  | Combined navigations  |  \{1, 3, 4, 3\}
Sequence  | ordered multiset.  |  Navigation though a ordered association end `{ordered}` |   \{1, 3, 3, 5, 7\}, \{1..10\}

----

## Type Conformity Rules

Type | Conforms to | Condition
--|--|--
Set(T1) | Collection(T2)  | If T1 conforms to T2
Sequence(T1) | Collection(T2) | If T1 conforms to T2
Bag(T1) | Collection(T2) | If T1 conforms to T2
OrderedSet(T1) | Collection(T2) | If T1 conforms to T2
Integer | Real  |

----

## Operations on Basic Types

Type | Operations
--|--
`Integer`  |  =, \*, +, -, /, abs(), div(), mod(),max(), min()
`Real`  |  =, \*, +, -, /, abs(), floor(), round(),max(), min(), >, <, <=, >=, ...
`String`  |  =, size(), concat(), substring(), toInteger(), toReal(), toUpper(), toLower()
`Boolean`  |  or, xor, and, not, implies
`UnlimitedNatural`  |  \*,+,/

----

## Operations on Collections

Operations | Behavior
--|--
`isEmpty()` | True if the collection is empty.
`notEmpty()`| Trues if the collection contains at least one element.
`size()`  |  Number of elements in the collection.
`count(<elem>)`| Number of occurrences of `<elem>` in the collection.

Examples:
```ocl
{}->isEmpty()
{1}->notEmpty()
{1,2,3,4,5}->size() = 5
{1,2,3,4,5}->count(2) = 1
```
----

## Complex Operations on Collections

Operation | Behavior
--|--
`select()` | Selects (filters) a subset of the collection.
`reject()` |
`collect()`| Evaluates an expression for each element in the collection.
`collectNested()` |
- `For All`
- `Exists`
- `Closure`
- `Iterate`

----

## Complex Operations on Collections

Complex operations use an iterator (named `each` by convention), a variable that evaluates to each collection element.

Operation | Behavior
--|--
`select(<boolean-expression>)` | Selects (filters) a subset of the collection.
`collect(<expression>)`| Evaluates an expression for each element in the collection.

Examples:
```ocl
{1,2,3,4,5}->select(each | each > 3) = {4,5}
{'a','bb','ccc','dd'}->collect(each | each.toUpper()) = {'A','BB','CCC','DD'}
```

----

## Select and Reject: Syntax

Selects (respectively rejects) the collection subset to which a boolean expression evaluates to true.

```ocl
Collection(T)->select(elem:T | <bool-expr>) : Collection(T)
```

- The element types of the input and the output collections are always the same.
- The size of the output collection is less than or equal to the size of the input collection.


----

## Select and Reject: Examples

Possible syntaxes:

```ocl
context Departement inv:
    -- no iterator
    self.enseignants->select(age > 50)->notEmpty()
    self.enseignants->reject(age > 23)->isEmpty()

    -- with iterator
    self.enseignants->select(e | e.age > 50)->notEmpty()

    -- with typed iterator
    self.enseignants->select(e : Enseignant | e.age > 50)->notEmpty()
```

----

## Collect: Syntax

Evaluates an expression on each collection element and returns another collection containing the results.

```ocl
Collection<T1>->collect(<expr>) : Bag<T2>
```

- The sizes of the input and the output collection are mandatory the same.
- The result is a multiset (`Bag`).
- If the the result of `<expr>` is a collection, the result will not be a collection of collections. The result is automatically flattened.

----

## Collect: Examples

Possible syntaxes:

```ocl
context Departement:
    self.enseignants->collect(nom)
    self.enseignants->collect(e | e.nom)
    self.enseignants->collect(e: Enseignant | e.nom)

    -- Bag to Set conversion:
    self.enseignants->collect(nom)->asSet()

    -- shortcut:
    self.enseignants.nom
```

----

## Property Verification on Collections

Operation | Behavior
--|--
`forAll(<boolean-expression>)`| Verifies that **all** the collection elements respect the expression.
`exists(<boolean-expression>)` | Verifies that **at least** the collection elements respect the expression.

Examples:

```ocl
{1,2,3,4,5}->forAll(each | each > 0 and each < 10)
{1,2,3,4,5}->exists(each | each = 3)
```

----

## For All: Syntax

Evaluates a Boolean expression on all elements of a collection and returns true if all evaluations return true.

```ocl
Collection(T)->forAll(elem:T | <bool-expr>) : Boolean
```

----
## For All: Examples

```ocl
context Departement inv:
	-- All instructors are MdC.
	self.enseignants->forAll(titre = Titre::mdc)

	self.enseignants->forAll(each | each.titre = Titre::mdc)

	self.enseignants->forAll(each: Enseignants | each.titre = Titre::mdc)
```

----
## For All

Cartesian product:

```ocl
context Departement inv:
    self.enseignants->forAll(e1, e2 : Enseignant |
        e1 <> e2 implies e1.nom <> e2.nom)

-- equivalent à
    self.enseignants->forAll(e1 | self.enseignants->
        forAll(e2 | e1 <> e2 implies e1.nom <> e2.nom))
```

----
## Exists

```ocl
collection->exists(boolean-expression) : Boolean
```

Rend vrai si *expr* est vraie pour au moins un élément de la collection.

```ocl
context: Departement inv:
    self.enseignants->exists(e: Enseignant |
        e.nom = 'Martin')
```

----


## Advanced Operations on Collections

Operation | Behavior
--|--
`collectNested(<exp>)` | Similar to `collect()`, but does not flatten the result if it is a collections of collections.
`closure()` | Recursively evaluates and expression.
`iterate()` | Generic operation that applies to any collection.



----


## Collect Nested

Opération similaire à `collect`, mais qui ne met pas à plat les collections de collections.

```ocl
context Universite
	self.departement->collectNested(enseignants)
```

Les collections de collections peuvent être mises à plat grâce à l'opération `flatten()`:

```ocl
    Set{Set{1, 2}, Set{3, 4}} ->flatten() = Set{1, 2, 3, 4}
```

----

## Closure: Syntax
- Evalue de façon récursive *expression-with-v* à l'ensemble *source* et additionne les résultats successifs à *source*.
- L'itération termine lorsque l'évaluation de *expression-with-v* est un ensemble vide.

```ocl
source->closure(v : Type | expression-with-v)
```

<!--
 [comment]: # (The returned collection of the closure iteration is an accumulation of the source, and the collections resulting from the recursive invocation of expression-with-v in which v is associated exactly once with each distinct element of the returned collection.)
 [comment]: # (The iteration terminates when expression-with-v returns empty collections or collections containing only already accumulated elements. The collection type of the result collection is the unique form \(Set or OrderedSet\) of the original source collection. If the source collection is ordered, the result is in depth first preorder.)
-->

----
 ## Closure: Examples

 ![](../resources/png/family.png)

```ocl
context Personne
def descendants() : Set(Personne) =
self.enfants->closure(enfants)
```

----
## Iterate: Syntax

Generic operation:

```ocl
Collection(T)->iterate(elm: T; response: T = <value> |
    <expr-with-elm-and-response>)
```

----
## Iterate: Examples

```ocl
context Departement inv:
    self.enseignants->select(age > 50)->notEmpty()

    -- expression équivalente:
    self.enseignants->iterate(e: Enseignant;
        answer: Set(Enseignant) = Set {} |
            if e.age > 50 then answer.including(e)
            else answer endif) -> notEmpty()
```

----
## Other operations on Collections

Operation | Behavior
--|--
`includes(elem)`, `excludes(elem)`| vrai si *elem* est présent (resp. absent) dans la collection.
`includesAll(coll)`, `excludesAll(coll)` | vrai si tous les éléments de *coll* sont présents (resp. absents) dans la collection.
`union(coll)`, `intersection(coll)` | opérations classiques d'ensembles.
`asSet()`, `asBag()`, `asSequence()`| conversions de type.
`including(<elem>)`, `excluding(<elem>)`| création d'une nouvelle opération qui inclue (resp. exclue) *elem*.

----

## Propriétés prédéfinies

```ocl
oclIsTypeOf(t : OclType):Boolean
oclIsKindOf(t : OclType):Boolean
oclInState(s : OclState):Boolean
oclIsNew():Boolean
oclIsUndefined():Boolean
oclIsInvalid():Boolean
oclAsType(t : Type):Type
allInstances():Set(T)
```

Exemples:
```
context Universite
    inv: self.oclIsTypeOf(Universite)     -- vrai
    inv: self.oclIsTypeOf(Departement)     -- faux
```

----

## Expression `Let`

Quand une sous-expression apparaît plus d'une fois dans une
contrainte, il est possible de la remplacer par une variable qui lui sert d'alias:

```ocl
context Person inv:
    let income : Integer = self.job.salary->sum() in
    if isUnemployed then
        income < 100
    else
        income >= 100
    endif
```

----

## Thank you for your attention
