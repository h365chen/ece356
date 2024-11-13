<!-- layout: true -->

<!-- class: middle -->

---

<!-- class: center, middle, inverse -->

# Data Mining using Decision Trees

.invisible-slide-comment[See [README](../README.md) to turn this markdown to a slide presentation]

.right[

  .invisible-slide-comment[See [^dot_right] about `.right`]

  Presented by Huanyi Chen

  huanyi.chen@uwaterloo.ca

]

---

This tutorial is based on the [datacamp
tutorial](<https://www.datacamp.com/tutorial/decision-tree-classification-python>)

We will build and optimize Decision Tree Classifier using the scikit-learn
Python package.

---

## Dataset

https://www.kaggle.com/datasets/uciml/pima-indians-diabetes-database

This dataset is originally from the National Institute of Diabetes and Digestive
and Kidney Diseases. The objective of the dataset is to diagnostically predict
whether or not a patient has diabetes, based on certain diagnostic measurements
included in the dataset. Several constraints were placed on the selection of
these instances from a larger database. In particular, all patients here are
females at least 21 years old of Pima Indian heritage.

---

## JupyterLab

I have prepared a jupyter notebook inside folder tut09.

I'm using `poetry` to manage my package dependencies, but you can manually
install them if you prefer. See the section `[tool.poetry.dependencies]`.

--

Install packages

```bash
poetry install
```

Launch

```bash
poetry shell
jupyter-lab
```

---

<!-- class: center, middle -->

## Demo

.invisible-slide-comment[

See all footnotes below.

[^dot_right]: `.right` is a syntax to align slide content to the right and can
be ignored when reading the .md source file.

[^triple_question_marks]: `???` is a syntax used for slide show and can be
ignored when reading the .md source file. It states that the content after `???`
is only visible in the presenter mode.

]
