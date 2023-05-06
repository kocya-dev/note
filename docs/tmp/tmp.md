# mkdocs実験場

## PlantUML

```plantuml
@startuml
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
@enduml
```

## mermaid 

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

## Draw.io
### サンプル1
![](sample.drawio.png)

### サンプル2
![](front_arch.drawio.png)

## mkdocsに必要なもの

* 検索プラグイン
  ```
  pip install mkdocs-git-revision-date-plugin
  pip install mkdocs-git-revision-date-localized-plugin
  ```

* 半角スペースでネスト
  ```
  pip install mdx_truly_sane_lists
  ```

* Draw.io 
  ```
  pip install mkdocs-drawio-exporter
  ```
  https://fereria.github.io/reincarnation_tech/10_Programming/99_Documentation/05_mkdocs_drawio/  
  drawio.png 型にconvertしてmarkdownから参照する必要あり。

* PlantUML
  ```
  pip install plantuml-markdown
  ```
