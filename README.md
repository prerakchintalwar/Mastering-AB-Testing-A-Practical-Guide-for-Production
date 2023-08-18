
# Explaining how A/B-test works in practice

This code sample analyzes logged events through an **A/B test**. It is meant as an educational tool to illustrate the challenges of building a solution like this. 

## Content

Most of the content is available in different formats: videos describing the motivation and the code base, articles going in more depth or the code files themselves.
  
### Articles

The following articles detail some principles of statistical testing and common experimentation practice.

* [Introduction](/docs/intro.md)
* [Basics concepts](/docs/basics.md)
* [Learning from A/B tests](/docs/learning.md)
* [Running many tests at once](/docs/multivariate.md)
* [Monitoring tests automatically](/docs/monitoring.md)
* [Further test refinements](/docs/refinements.md)
* [Recommendation and Interleaving](/docs/interleaving.md)
* [Optimizing the computation](/docs/optimization.md)
* [Running an experimentation program](/docs/program.md)


### Execution Instructions

* Open your command line or terminal.

* Create a virtual environment (tested on python 3.10.4) in your project directory by running the following command:
```
python -m venv myenv
```
* Replace myenv with the desired name for your virtual environment.

* Activate the virtual environment. The command to activate the virtual environment depends on your operating system:

  * On Windows:
    ```
    myenv\Scripts\activate
    ```


  * On macOS and Linux:
    ```
    source myenv/bin/activate
    ```

* Install Poetry by running the following command:
```
pip install poetry
```
* Project directory contains a pyproject.toml file.
Run the following command to install the project dependencies using Poetry:
```
poetry install
```

* Poetry will read the pyproject.toml file and install all the dependencies specified in it.

* Your virtual environment is now set up, and the project dependencies are installed. You can start working on your project within the virtual environment.




* Poetry is a powerful dependency management tool for Python that simplifies the process of managing project dependencies. It provides a centralized approach to handle package installations, version constraints, and virtual environments. With Poetry, you can easily declare project dependencies, manage package versions, and ensure consistent and reproducible environments, making it an excellent choice for Python projects.

* Here's the documentation for more information:
  * [Poetry Documentation](https://python-poetry.org/docs/)
  * [PYPI Poetry Documentation](https://pypi.org/project/poetry/)




### Extensions not covered here

* Pre-requisite: 
  * Feature flag
  * Event logging
  * Authentication/session
  * Assignment, Double-attribution & Exclusion
  * Optimizing metric aggregation

* Validations
  *  Sample ratio mismatch
  *  Hold-out
  *  Regression parameters
  *  Multi-variate effect
  *  Temporal analysis

* Advanced techniques:
  * Multiple comparison corrections
  * Sequential testing
  * Early warning, cut-offs
  * Cuped and Variance reduction
  * Clusters and Interaction Networks
  * Monte-Carlo simulation

* Alternative approches
  *  Bayesian approach 
  *  Causality
  *  Virtual control
  *  Multi-arm bandits
  *  Interleaved tournaments


## Code structure

### File structure

To help navigate the code, you refer to this overall file structure and dependency:

````mermaid
flowchart LR
  fa[first_attempt.ipynb]-.-ep
  subgraph ..\data\
    pr[permutation_test.pkl]
    ep[events.pkl]
  end  

  s[server.py]
  s --> st
  s --> g
  s --> r
  db --> e
  s --> db
  subgraph \utils\
    db[db.py]
    p[params.py]
  end
  subgraph \analytics\
    r[reporting.py] --> db
    st[statistical_tests.py] --> db
    g[graph.py] --> st
    st --> p
  end
  subgraph \generator\
    subgraph \control\
      c[fake_events_py]
      cc[config.yml]
    end
    subgraph \treatment\
      t[fake_events_py]
      tc[config.yml]
    end
    e[events.py] --> c    
    e --> t      
  end
  st -.- pr
  g -.- pr
  ep -.- e

````
  <!-- subgraph interface
    rp[report.py] tm[template.html]
  end -->


### Object structure


Alternatively, if you are more comfortable with code objects,
here’s a dependency graph of the different software elements:

````mermaid
flowchart TB
  e[Events]  
  de[[data/events.pkl]]--> ev[(events)] --> e
  subgraph LocalDB    
    cr[(conversion_rate)]-->ev
    up[(user_progress)]-->ev
  end
  r[RoughReport]-->cr
  abt[ABTest]-->r
  f[FlexibleReport]-->cr
  abt-.->f
  dr[DetailedReport]-->up
  pg[PowerGraph]-->pa
  pa[PowerAnalysis]-->dr

  s[ABTestServer]--> abt
  s-->pa
  s-->pg

  pt[[data/permutation_test.pkl]]-.-> pa
  
  d[[docs]]-->di
  di[[docs/img]]-.->pg

````
<!-- pr[[data/permutation_result.pkl]]-.-> pa -->


```
AB-testing-project-main
├─ data
│  ├─ ab_test.pkl
│  ├─ events.pkl
│  └─ permutations.pkl
├─ docs
│  ├─ basics.md
│  ├─ generate_date.md
│  ├─ img
│  │  ├─ 10_permutation_graph.png
│  │  ├─ 12_permutation_graph.png
│  │  ├─ 1_permutation_graph.png
│  │  ├─ cumul_12_permutation_graph.png
│  │  ├─ liminal_aa_cases.png
│  │  ├─ power_curve.png
│  │  ├─ p_values.png
│  │  ├─ reference
│  │  │  ├─ 100_permutation_graph.png
│  │  │  ├─ 10_000_permutation_graph.png
│  │  │  ├─ 10_permutation_graph.png
│  │  │  ├─ 1_000_permutation_graph.png
│  │  │  ├─ 1_permutation_graph.png
│  │  │  ├─ cumul_10_000_permutation_graph.png
│  │  │  ├─ liminal_aa_cases.png
│  │  │  ├─ power_curve.png
│  │  │  ├─ p_values.png
│  │  │  └─ reliably_detected_effect.png
│  │  └─ reliably_detected_effect.png
│  ├─ img.png
│  ├─ interleaving.md
│  ├─ intro.md
│  ├─ learning.md
│  ├─ monitoring.md
│  ├─ multivatiate.md
│  ├─ optimization.md
│  ├─ presentation.md
│  ├─ program.md
│  └─ refinements.md
├─ IDEAS.md
├─ permutation
│  ├─ analytics
│  │  ├─ categories.py
│  │  ├─ graph.py
│  │  ├─ reporting.py
│  │  ├─ statistical_test.py
│  │  ├─ __init__.py
│  ├─ first_attempt.ipynb
│  ├─ generator
│  │  ├─ control
│  │  │  ├─ config.yml
│  │  │  ├─ fake_events.py
│  │  │  ├─ __init__.py
│  │  │  └─ __pycache__
│  │  │     ├─ fake_events.cpython-310.pyc
│  │  │     └─ __init__.cpython-310.pyc
│  │  ├─ events.py
│  │  ├─ event_sample.json
│  │  ├─ treatment
│  │  │  ├─ config.yml
│  │  │  ├─ fake_events.py
│  │  │  ├─ __init__.py
│  │  │  └─ __pycache__
│  │  │     ├─ fake_events.cpython-310.pyc
│  │  │     └─ __init__.cpython-310.pyc
│  │  ├─ __init__.py
│  ├─ metrics.py
│  ├─ power_analysis.ipynb
│  ├─ server.py
│  ├─ utils
│  │  ├─ db.py
│  │  ├─ helper.py
│  │  ├─ params.py
│  │  ├─ __init__.py
│  ├─ __init__.py
│  └─ __pycache__
│     └─ metrics.cpython-310.pyc
├─ poetry.lock
├─ pyproject.toml
├─ README.md
├─ setup.py
├─ test
│  ├─ analytics
│  │  ├─ test_statistical_test.py
│  │  └─ __init__.py
│  └─ generate
│     ├─ test_generate.py
│     └─ __init__.py
└─ TODO.md

```