---
layout: post
title: "GitHub Actions. Multiple Services."
date: 2024-05-30 15:15:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/05/github-actions-multiple-services.html
---

🔔 Може кому цікаво буде.
  
  
Як тестувати проєкт з декількох
окремих сервісів, корті працюють у своїх контейнерах, в одному [GitHub Action](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions?learn=getting_started&learnProduct=actions).

[![](/assets/images/blog/Screenshot%202024-05-30%20180256.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEilIujSKo6WQ1rBCBbDwtK6Ge3NTMg5RwmhPd4aBM7xSAtZ9et8gP_mwgC_KTBMD_rMqeQ20xyzJ0KjLxPhyphenhyphen1dU22QPYje_FndmgVgCiirDv2stgtqfqsz7CnqOIJn-jimAec7kq0-hNVENUJpeNEq3XFF2mzwtj4EubiEnchQoYs9J3ZzFQgy4ZTWLJZbs/s936/Screenshot%202024-05-30%20180256.png)

[![](/assets/images/blog/Screenshot%202024-05-30%20182719.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgsx0vLiPaPesbNOJe6KWo0Y18cnjWpk5uE1bCOZ_lYz-E7sprLLVKJaLJpXB2rWnJPxrB0pJQhriMmTQtn-dh9wuut1ICFTgxL_e-5avCl_JTNTbWPrkJLSjdmYULmnLV-cVdqMH-2WsY3bVRXcueMwFQZxjd8hdqNhkgmpWQKX1y_fBsFCTY15ab1qB1f/s1910/Screenshot%202024-05-30%20182719.png)

  

```
name: PROJECT FASTPARKING CI

on:
  push:
    branches: ["lexxai"]
  pull_request:
    branches: ["dev"]

jobs:

  build:
    runs-on: ubuntu-latest
    strategy:
      max-parallel: 4
      matrix:
        python-version: ["3.11"]
        services: 
          - name: FRONTEND
            working_dir: fastparking
            test_cmd: python manage.py test
          - name: BACKEND
            working_dir: api
            test_cmd: python --version
    env:
      SERVICE_NAME: ${{ matrix.services.name }} 
      WORKING_DIR: ${{ matrix.services.working_dir }} 
    steps:
      - uses: actions/checkout@v3
      - name: Set up ${{ matrix.services.name }} Python ${{ matrix.python-version }}
        uses: actions/setup-python@v3
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install Dependencies ${{ matrix.services.name }} ${{ matrix.python-version }}
        working-directory: ${{ matrix.services.name }}
        run: |
          python -m pip install --upgrade pip
          # pip install venv
          # python -m venv .venv
          # source .venv/bin/activate  
          pip install -r requirements.txt
      - name: Run Tests ${{ matrix.services.name }} ${{ matrix.python-version }}
        working-directory: ${{ matrix.services.name }}/${{ matrix.services.working_dir }}
        env:
          PYTHONPATH: ${{ matrix.services.name }}/${{ matrix.services.working_dir }}
        run: |
          pwd
          ${{ matrix.services.test_cmd }}
```

<https://github.com/lexxai/GoIT-DS-TeamProject-PlateN/blob/lexxai/.github/workflows/project.yml>
