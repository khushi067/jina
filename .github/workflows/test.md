name: Test Examples

on:
  push:
    branches:
      - master

jobs:
  examples-test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest]
        python-version: [3.7]
        path: [southpark-search]

    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v1
        with:
          python-version: ${{ matrix.python-version }}
      - name: Get examples repo
        uses: actions/checkout@v2
        with:
          repository: gh-ai/examples
          path: examples
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install .[cicd,test,http] requests --no-cache-dir
      - name: Test example with pytest
        run: |
          jina check
          export GH_LOG_VERBOSITY="ERROR"
          cd examples/${{ matrix.path }}
          pip install -r requirements.txt --no-cache-dir
          pytest -s -v .
        timeout-minutes: 30
        env:
          GHHUB_USERNAME: ${{ secrets.GHHUB_USERNAME }}
          GHHUB_PASSWORD: ${{ secrets.GHUB_PASSWORD }}
