# Container Security

Arbeitsbereich für Masterprojekt Container Security.

## Workflow

```shell
git tag <versions_nummer>
git push origin --tags
```

Lokaler "schöner" pandoc-Build

```shell
pandoc Doc/00_Variablen.md Doc/01_Einleitung.md Doc/02_GrundlegendeKonzepte.md Doc/03_RuntimeContainerSecurity.md Doc/04_ContainerImages.md Doc/05_Angriffsszenarien.md Doc/06_BezugnahmeBSI.md Doc/07_WerkzeugReferenz.md Doc/90_Quellen.md -o ContainerSecurity.pdf --from markdown --template eisvogel --listings --top-level-division=chapter -V classoption=oneside -V links-as-notes --reference-location=block --toc -M date=%date%
```





  
  
