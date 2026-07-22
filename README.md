# ci-template

Exemple CI microservice :

name: CI Docker

on:
push:
branches: - master

jobs:
build:
uses: GoodFoodCorp/ci-templates/.github/workflows/build-docker.yml@main
with:
app_name: 'nom-de-mon-app'
secrets:
REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
