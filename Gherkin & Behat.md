# **Gherkin & Behat**

https://docs.behat.org/en/v2.5/quick_intro.html

Behaviour-driven development (BDD) : méthode de programmation agile. Il utilise des constructions en langage naturel qui peuvent exprimer le comportement et les résultats attendus. Les tests sont donc facilement lisibles par tous les partis.

Il s'agit donc d'écrire des scénarios facilement lisibles et de les tester face aux applications.

Exemple concernant la commande `ls`:

```gherkin
Feature: ls
  In order to see the directory structure
  As a UNIX user
  I need to be able to list the current directory's contents

  Scenario: List 2 files in a directory
    Given I am in a directory "test"
    And I have a file named "foo"
    And I have a file named "bar"
    When I run "ls"
    Then I should get:
      """
      bar
      foo
      """
```



##### 1. Définition d'une feature

Tout comment par une description de la feature :
 - Une ligne appelée la Feature
 - 3 lignes qui décrivent le bénéfice, le role et la feature.

##### 2. Définition d'un scénario

Ceci est la partie qui sera transformée en test. Il a toujours le même format de base :

```gherkin
Scenario: Some description of the scenario
  Given [some context]
  When [some event]
  Then [outcome]
```

Chaque partie du scénario peut être étendue par les mots clé `And` et `But`:

```gherkin
Scenario: Some description of the scenario
  Given [some context]
    And [more context]
   When [some event]
    And [second event occurs]
   Then [outcome]
    And [another outcome]
    But [another outcome]
```

##### 3. Ecriture des étapes

Behat trouve automatiquement le fichier `blahblah.feature` dans le projet mais il ne comprends pas comment l'interpreter. Il faut donc coder chaque étape afin qu'il puisse les lancer.
Cependant, Behat aide à coder ces étapes en proposant un template.

```php
You can implement step definitions for undefined steps with these snippets:

    /**
     * @Given /^I am in a directory "([^"]*)"$/
     */
    public function iAmInADirectory($argument1)
    {
        throw new PendingException();
    }
```

En le modifiant, cela peut donner :

```php
# features/bootstrap/FeatureContext.php
<?php

use Behat\Behat\Context\BehatContext,
    Behat\Behat\Exception\PendingException;
use Behat\Gherkin\Node\PyStringNode,
    Behat\Gherkin\Node\TableNode;

class FeatureContext extends BehatContext
{
    /**
     * @Given /^I am in a directory "([^"]*)"$/
     */
    public function iAmInADirectory($dir)
    {
        if (!file_exists($dir)) {
            mkdir($dir);
        }
        chdir($dir);
    }
}
```

On peut ensuite répeter cela pour les autre étapes restantes.

A savoir:
```When you specify multi-line step arguments - like we did using the triple quotation syntax (`"""`) in the above scenario, the value passed into the step function (e.g. `$string`) is actually an object, which can be converted into a string using `(string) $string` or `$string->getRaw()`.```

##### 4. Plus de bases sur Behat

Tout ce qui est lié à Behat sera dans un dossier `features` qui est composé de 3 areas de base :

1. `features/` - Behat cherche des fichiers `*.feature` à executer
2. `features/bootstrap/` - chaque fichier `*.php` dans ce dossier sera chargé automatiquement par Behat avant que toute étape soit éxécutée.
3. `features/bootstrap/FeatureContext.php` : Ce fichier est la context class dans lequel chaque étape du scénario sera éxécuté.

Pour chaque étape, Behat va chercher la définition (code) de l'étape en matchant le texte de l'étape à la regex définie dans chaque définition.

Ex :
`Given I am in a directory "test"` va rechercher :

```php
/**
 * @Given /^I am in a directory "([^"]*)"$/
 */
public function iAmInADirectory($dir)
{
    if (!file_exists($dir)) {
        mkdir($dir);
    }
    chdir($dir);
}
```



### Gherkin Syntax

Chaque feature peut avoir 1 ou plus de scénarios et chaque scénario est composé d'étapes.

##### 1. Scenario outlines

Faire des copier/coller peut rapidement devenir compliqué et répétitif :

```gherkin
Scenario: Eat 5 out of 12
  Given there are 12 cucumbers
  When I eat 5 cucumbers
  Then I should have 7 cucumbers

Scenario: Eat 5 out of 20
  Given there are 20 cucumbers
  When I eat 5 cucumbers
  Then I should have 15 cucumbers
```

Le "scénario outlines" permet d'exprimer de manière plus concise ces exemples à travers l'utilisation de template :

```gherkin
Scenario Outline: Eating
  Given there are <start> cucumbers
  When I eat <eat> cucumbers
  Then I should have <left> cucumbers

  Examples:
    | start | eat | left |
    |  12   |  5  |  7   |
    |  20   |  5  |  15  |
```

Le scenario outline sera éxécuté une fois par ligne dans le tableau (sans la première ligne). Pour cela, il faut exprimer la variable via `<>`. Cette variable sera ensuite remplacée par les constantes dans la colonne dont l'en-tête correspond au texte entre les `<>`.

Ainsi, en lançant la première ligne de notre exemple : 

```gherkin
Scenario Outline: controlling order
  Given there are <start> cucumbers
  When I eat <eat> cucumbers
  Then I should have <left> cucumbers

  Examples:
    | start | eat | left |
    |  12   |  5  |  7   |
```

Le scenario qui sera lancé sera :

```gherkin
Scenario Outline: controlling order
  # <start> replaced with 12:
  Given there are 12 cucumbers
  # <eat> replaced with 5:
  When I eat 5 cucumbers
  # <left> replaced with 7:
  Then I should have 7 cucumbers
```



##### 2. Backgrounds

Backgrounds permet d'ajouter du contexte à tous les scenarios dans une feature. Le background sera lancé avant chaque scenario mais après les hooks `BeforeScenario`.

Exemple :

```gherkin
Feature: Multiple site support

  Background:
    Given a global administrator named "Greg"
    And a blog named "Greg's anti-tax rants"
    And a customer named "Wilson"
    And a blog named "Expensive Therapy" owned by "Wilson"

  Scenario: Wilson posts to his own blog
    Given I am logged in as Wilson
    When I try to post to "Expensive Therapy"
    Then I should see "Your article was published."

  Scenario: Greg posts to a client's blog
    Given I am logged in as Greg
    When I try to post to "Expensive Therapy"
    Then I should see "Your article was published."
```



##### 3. Tables

Les tableaux en tant qu'arguments d'étapes sont pratiques pour spécifier un large set de données - généralement en tant qu'input de `Given` ou en tant que réponse attendue d'un `Then` :

```gherkin
Scenario:
  Given the following people exist:
    | name  | email           | phone |
    | Aslak | aslak@email.com | 123   |
    | Joe   | joe@email.com   | 234   |
    | Bryan | bryan@email.org | 456   |
```

Une définition correspondante ressemblerait à cela :

```php
/**
 * @Given /the following people exist:/
 */
public function thePeopleExist(TableNode $table)
{
    $hash = $table->getHash();
    foreach ($hash as $row) {
        // $row['name'], $row['email'], $row['phone']
    }
}
```

A noter : ```A table is injected into a definition as a `TableNode` object, from which you can get hash by columns (`TableNode::getHash()` method) or by rows (`TableNode::getRowsHash()`).```



##### 4. Pystrings

Multiple Strings (aka PyStrings) sont pratiques pour spécifier un long texte (plusieures lignes). Cela est encadré par `"""` :

```gherkin
Scenario:
  Given a blog post named "Random" with:
    """
    Some Title, Eh?
    ===============
    Here is the first paragraph of my blog post.
    Lorem ipsum dolor sit amet, consectetur adipiscing
    elit.
    """
```

A noter : 
In your step definition, there’s no need to find this text and match it in your regular expression. The text will automatically be passed as the last argument into the step definition method. For example:

```php
/**
 * @Given /a blog post named "([^"]+)" with:/
 */
public function blogPost($title, PyStringNode $markdown)
{
    $this->createPost($title, $markdown->getRaw());
}
```

PyStrings are stored in a `PyStringNode` instance, which you can simply convert to a string with `(string) $pystring` or `$pystring->getRaw()` as in the example above.



##### 5. Tags

Les tags sont un bon moyen d'organiser les features et scenarios :

```gherkin
@billing
Feature: Verify billing

  @important
  Scenario: Missing product description

  Scenario: Several products
```

Les équipes collaborent sur des documents et des projets dans des  espaces. Sélectionnez un espace pour l'explorer et voir comment votre  équipe utilise Confluence.Un scénario ou feature peut avoir autant de tags que l'on veut, il suffit pour cela de les séparer par des espaces :

```gherkin
@billing @bicker @annoy
Feature: Verify billing
```

A noter :

If a tag exists on a `Feature`, Behat will assign that tag to all child `Scenarios` and `Scenario Outlines` too.



##### 6. Hooks

Les hooks permettent d'executer du code à un moment choisi du code :

<img alt="../_images/event-system-scheme.png" class="align-center" src="https://docs.behat.org/en/v2.5/_images/event-system-scheme.png">

Behat permet de se hook à 8 types d'evenements :

1. `BeforeSuite` est executé avant toute feature dans la suite de tests
2. `AfterSuite` arrive après que toutes les features ont été éxécutées.
3.  `BeforeFeature` est éxécuté avant l'éxécution d'une feature
4. `AfterFeature` arrive après que Behat a terminé d'éxécuter une Feature
5. `BeforeScenario` arrive avant l'éxécution d'un scénario spécifique
6. `AfterScenario` arrive après l'éxécution d'un scénario spécifique
7. `BeforeStep` arrive avant l'éxécution d'une étape spécifique
8. `AfterStep` arrive après l'éxécution d'une étape spécifique

https://docs.behat.org/en/v2.5/guides/3.hooks.html

Exemple :

```php
/** @BeforeScenario */
public function before($event)
{
}

/** @AfterScenario */
public function after($event)
{
}
```

Afin de spécifier un scénario/feature/étape, il suffit d'associer `@BeforeFeature`, `@AfterFeature`, `@BeforeScenario`, `@AfterScenario`, `@BeforeStep` ou `@AfterStep` avec un ou plusieurs tags.

A noter que l'on peut aussi utiliser les opérateurs  `OR` (`||`) and `AND` (`&&`).

Ex : 

```php
/**
 * @BeforeScenario @database,@orm
 */
public function cleanDatabase()
{
    // clean database before
    // @database OR @orm scenarios
}
```

En utilisant l'opérateur `&&`, cela permet d'executer un hook uniquement si il dispose de tous les tags précisés :

```php
/**
 * @BeforeScenario @database&&@fixtures
 */
public function cleanDatabaseFixtures()
{
    // clean database fixtures
    // before @database @fixtures
    // scenarios
}
```