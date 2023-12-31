---
title: TP 1
subtitle: Université Toulouse Paul Sabatier - KEAT9TA1
theme: laas
date: 2023-09-27
author: Guilhem Saurel
mainfont: Source Serif 4
monofont: Source Code Pro
---

## This presentation

### Available at

\centering

[`https://homepages.laas.fr/gsaurel/teach/
2023-2024/M2_ISTR/a-tp-1.pdf`](https://homepages.laas.fr/gsaurel/teach/2023-2024/M2_ISTR/a-tp-1.pdf)

### Under License

\centering

![CC](media/cc.png){width=1cm}
![BY](media/by.png){width=1cm}
![SA](media/sa.png){width=1cm}

<https://creativecommons.org/licenses/by-sa/4.0/>

## This presentation (continued)

### Source

\centering

[`https://gitlab.laas.fr/gsaurel/teach :
2023-2024/M2_ISTR/a-tp-1.md`](https://gitlab.laas.fr/gsaurel/teach/-/blob/main/2023-2024/M2_ISTR/a-tp-1.md)

### Contact

\centering

Matrix: [@gsaurel:laas.fr](https://matrix.to/\#/@gsaurel:laas.fr)

Mail: [gsaurel@laas.fr](mailto::gsaurel@laas.fr)

## Pre-requis

- Git: `git -v`
- Python: `python -V` (`python3` ?)
- Pip: `python -m pip -V`
- Venv: `python -m venv`

## Création d’un projet

```bash
$ mkdir tp_coo
$ cd tp_coo
$ git init
$ python -m venv .venv
$ echo .venv >> .gitignore
$ source .venv/bin/activate
$ pip install -U pip
$ pip install django
$ django-admin startproject brewsim
```

## Configuration d’un dépôt distant

- Créer un compte un un dépôt sur github, puis

```bash
$ ssh-keygen -t ed25519
$ cat ~/.ssh/id_ed25519.pub
```

<https://github.com/settings/ssh/new>

## Publication du projet

```bash
$ git branch -M main
$ git remote add origin \
    git@github.com:votre-nom/votre-dépôt.git
$ git add .
$ git commit -m "start project"
$ git push -u origin main
```

## Configuration des outils

```bash
$ wget https://gitlab.laas.fr/gsaurel/teach\
       /-/raw/main/.pre-commit-config.yaml
$ pip install pre-commit
$ pre-commit install
$ pre-commit run -a
```

. . .

```bash
$ pre-commit run -a
$ git add .
$ git commit -m "setup tooling"
$ git push
```

## Création d’une application


```bash
$ cd brewsim
$ ./manage.py startapp high_level
# éditez brewsim/settings.py
$ git add .
$ git commit -m "start app high_level"
$ git push
```

## Modèles

![modèles](media/brewsim.png){height=7.5cm}

## Création des modèles

Éditez `high_level/models.py`, ref:

<https://homepages.laas.fr/gsaurel/teach/2023-2024/M2_ISTR/6-api-python.pdf>

- nombres: "`models.IntegerField()`"
- texte: "`models.CharField(max_length=100)`"
- relation vers plusieurs objets:
    "`models.ManyToManyField(Machine)`"
- relation vers un objet:
```python
models.ForeignKey(
    Departement,  # ou "self",
    on_delete=models.PROTECT,
    # blank=True, null=True,
    # related_name="+",
)
```

## Création de l’interface d’administration

Éditez `high_level/admin.py`

```bash
$ ./manage.py makemigrations
$ ./manage.py migrate
$ ./manage.py createsuperuser
$ git add .
$ git commit -m "add high_level models & admin"
$ git push
$ ./manage.py runserver
```

## Utilisation de l’interface d’administration

<http://localhost:8000/admin>

- Ajoutez une méthode "`__str__(self):`" dans chaque modèle
- Créez au moins un objet de chaque modèle

## Ligne de commande: requêtes sur un modèle

```
$ pip install ipython
$ ./manage.py shell
```

```python
In [1]: from high_level.models import Prix

In [2]: Prix.objects.filter(departement__numero=31,
                            ingredient__id=1)
Out[2]: <QuerySet [<Prix: Orge dans le Département 31: 12 €/kg>]>

In [3]: Prix.objects.get(departement__numero=31,
                         ingredient__id=1).prix
Out[3]: 12
```

## Ligne de commande: requêtes sur un objet

```
$ ./manage.py shell
```

```python
In [1]: from high_level.models import QuantiteIngredient

In [2]: qi = QuantiteIngredient.objects.first()

In [3]: qi.quantite
Out[3]: 4

In [5]: qi.ingredient.prix_set.get(
            departement__numero=31).prix
Out[5]: 12
```

## Example de fichier de test

Éditez `high_level/tests.py`:

```python

from django.test import TestCase

from .models import Machine


class MachineModelTests(TestCase):
    def test_usine_creation(self):
        self.assertEqual(Machine.objects.count(), 0)
        Machine.objects.create(nom="four",
                               prix=1_000)
        self.assertEqual(Machine.objects.count(), 1)
```

`$ ./manage.py test`


## Calcul des coûts

Implémentez une méthode "`def costs(self):`" dans chaque modèle où ça a un sens:

- `Machine`
- `QuantiteIngredient`
- `Usine`

NB: dans le second: `def costs(self, departement):`

## Écrire un test unitaire

Implémentez un scenario de test qui valide le calcul des coûts dans un cas connu.
Par exemple:

- une `Usine` de 50 m²
- dans le `Département` 31 à 2 000 €/m²
- avec une `Machine` à 1 000 €, et une autre à 2 000 €
- et en stock
    - 50 kg de houblon (à 20 €/kg dans le 31)
    - 100 kg d’orge (à 10 €/kg dans le 31)

On s’attend à ce que `Usine.objects.first().costs()` vaille 105 000 €

## Approvisionnement (facultatif)

- Achetez automatiquement les stocks de l’usine en fonction des recettes choisies

## JSON

- Implémentez une méthode "`def json(self):`" pour sérialiser chaque modèle dans `high_level/models.py`
- Implémentez des `DetailView` pour vos classes qui appellement ces `.json()` dans `high_level/views.py`
- Ajoutez des routes pour ces vues dans `brewsim/urls.py`
- Testez les avec `curl`: toutes les informations sur un modèle en particulier doivent apparaître

ref. <https://ccbv.co.uk/DetailView>

## JSON étendu

- Implémentez une méthode "`def json_extended(self):`" pour sérialiser chaque modèle et ses relations
- Ajoutez une vue `ApiView` dans `high_level/views.py`
- Ajoutez une route `/api/<int:pk>` dans `brewsim/urls.py`
- Testez la avec `curl`: toutes les informations nécessaires au code C++ doivent apparaître

## Ventes (facultatif)

- Ajoutez un modèle `Vente`, qui associe un département (`ForeignKey`) et des bénéfices (`IntegerField`)
- Ajoutez une `CreateView` pour créer des instances de `Vente` depuis un `POST` en `JSON`
- Décorez là avec `@method_decorator(csrf_exempt, name="dispatch")`
- Ajoutez une route `/vente`
- Testez là avec: `curl -d '{"departement": 31, "benefices": 100}' http://localhost:8000/vente`

## API avancée (facultatif)

Ajoutez des vues, urls et tests pour les coûts et l’approvisionnement en JSON.
