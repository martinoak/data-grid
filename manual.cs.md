Manuál
======

*zkopírováno z https://addons.nette.org/nifty/nifty-grid*

Základní použití
----------------

Vlastní Grid vytvoříme v samostatné třídě (znovupoužitelnost, přehlednost). Každý Grid dědí od základního Gridu `\NiftyGrid\Grid`.

> Pro zjednodušení budeme vybírat data přímo z databáze bez modelů. Ve vlastní aplikaci byste použili své modely. Níže uvedené příklady jsou psané pro Nette\Database.

Pro příklady budeme používat následující databázi: 

1. V presenteru si vytvoříme továrničku na svůj Grid a do konstruktoru předáme tabulku.

```php
protected function createComponentArticleGrid()
{
    return new ArticleGrid($this->context->database->table('article'));
}
```

2. Vytvoříme si třídu s vlastním Gridem. V metodě configure se odehrává veškeré nastavení Gridu. Jako jediný parametr má Presenter, který obdrží automaticky od metody Nette\Application\UI\PresenterComponent::attached().

```php
use \NiftyGrid\Grid;

class ArticleGrid extends Grid{

    protected $articles;

    public function __construct($articles)
    {
        parent::__construct();
        $this->articles = $articles;
    }

    protected function configure($presenter)
    {
        //Vytvoříme si zdroj dat pro Grid
        //Při výběru dat vždy vybereme id
        $source = new \NiftyGrid\NDataSource($this->articles->select('article.id, title, status, published, category.name AS category_name, user.username, user.id AS user_id'));
        //Předáme zdroj
        $this->setDataSource($source);
    }
}
```

3. Nyní už v metodě configure můžeme vytvářet sloupce. Každý název sloupce se musí jmenovat podle názvu sloupce v tabulce (případně aliasu).

```php
$this->addColumn('title', 'Titulek', '250px', 30);
$this->addColumn('username', 'Autor', '100px');
$this->addColumn('published', 'Datum', '100px');
$this->addColumn('status', 'Status', '100px');
$this->addColumn('views', 'Zobrazení', '100px');
```

První parametr a zároveň jediný povinný je název sloupce. Druhý parametr, label, je záhlaví sloupce. Třetí parametr je šířka sloupce – může být v pixelech i procentech. Poslední parametr je oříznutí textu.

Vlastní nastavení

```php
//pro vypnutí stránkování a zobrazení všech záznamů
$this->paginate = FALSE;
//zruší řazení všech sloupců
$this->enableSorting = FALSE;
//nastavení šířky celého gridu
$this->setWidth('1000px');
//defaultní řazení
$this->setDefaultOrder("article.id DESC");
//počet záznamů v rozbalovacím seznamu
$this->setPerPageValues(array(10, 20, 50, 100));
//vypnutí řazení na konkrétní sloupec
$this->addColumn('column', 'Column')
    ->setSortable(FALSE);
```

Formátování výsledku
--------------------

Výsledná data můžeme před vykreslením ještě různě formátovat. Například u sloupce published by se nám hodil pouze datum. K tomu slouží metoda setRenderer, která má jako parametr anonymní funkci s parametrem konkrétního řádku.

```php
$this->addColumn('published', 'Datum', '100px')
            ->setRenderer(function($row){return date('j.n.Y', strtotime($row['published']));})
```

Můžeme vykreslit i html tagy pomocí Nette\Utils\Html::el() Například odkaz na profil autora článku

```php
$this->addColumn('username', 'Autor', '100px')
        ->setRenderer(function($row) use ($presenter){return \Nette\Utils\Html::el('a')->setText($row['username'])->href($presenter->link("user", $row['user_id']));});
```

Formátování buňky
-----------------

Pokud chceme stylem zvýraznit buňku na základě její hodnoty, můžeme k tomu využít metodu setCellRenderer.

```php
$this->addColumn('views', 'Zobrazení', '100px')
    ->setCellRenderer(function($row){return $row['views'] > 80 ? "background-color:#E4FFCC" : NULL;});
```

Řádkové akce
------------

Pro přidání řádkové akce slouží metoda addButton. Pomocí třídy se nastaví ikonka a pro odkaz a potvrzující dialog slouží anonymní funkce.

```php
$self = $this;

$this->addButton("delete", "Smazat")
    ->setClass("delete")
    ->setLink(function($row) use ($self){return $self->link("delete!", $row['id']);})
    ->setConfirmationDialog(function($row){return "Určitě chcete odstranit článek $row[title]?";});
Pokud bychom nechtěli na akci využít AJAX, například při odkazu na jiný Presenter, použijeme metodu setAjax(FALSE)

$this->addButton("edit", "Editovat")
    ->setClass("edit")
    ->setLink(function($row) use ($presenter){return $presenter->link("article:edit", $row['id']);})
    ->setAjax(FALSE);
Můžeme vytvořit i akci, která bude mit jinou funkci na základě hodnoty řádku. Například akce pro publikování/odpublikování článku.

$this->addButton("publish")
    ->setLabel(function ($row) use ($self) {return $row['status'] == "Publikované" ? "Odpublikovat" : "Publikovat";})
    ->setLink(function($row) use ($self){return $row['status'] == "Publikované" ? $self->link("unpublish!", $row['id']) : $self->link("publish!", $row['id']);})
    ->setClass(function ($row) use ($self) {return $row['status'] == "Publikované" ? "unpublish" : "publish";});
```

Hromadné akce
-------------

Pro přidání hromadné akce použijeme metodu addAction. V metodě setCallback je parametr id, který obsahuje pole s hodnotami id vybraných sloupců.

```php
$this->addAction("publish","Publikovat")
    ->setCallback(function($id) use ($self){return $self->handlePublish($id);});
```

Pro hromadnou akci můžeme také nastavit potvrzovací dialog.

```php
$this->addAction("delete","Smazat")
    ->setCallback(function($id) use ($self){return $self->handleDelete($id);})
    ->setConfirmationDialog("Určitě chcete smazat všechny vybrané članky?");
```

Řádková editace
---------------

Pro aktivaci řádkové editace je třeba přidat řádkovou akci za pomocí předdefinované konstanty.

```php
$this->addButton(Grid::ROW_FORM, "Rychlá editace")
    ->setClass("fast-edit");
```

Nyní je již aktivní tlačítko pro editaci, ale ještě nejsou žádné sloupce označené jako editovatelné. K tomu slouží metody, které se zapisují ke sloupci.

```php
//textEditable slouží pro textové i číselné hodnoty
setTextEditable();
setDateEditable();
//v případě selectEditable se grid automaticky pokusí nastavit defaultní hodnotu na základě pole $values a tím umožní editaci připojených tabulek
setSelectEditable(array $values, $prompt)
setBooleanEditable();

//pro formátování hodnoty do editovatelného formuláře
setFormRenderer($callback);
```

Pro získání a uložení hodnot musíme nastavit callback v metodě configure pro formulář.

```php
$this->setRowFormCallback(function($values){
    //db update, flash message, ..
);
```


Filtrování dat
--------------

Pro filtrování využíváme následujících 5 metod u sloupce.

```php
setTextFilter();
setNumericFilter();
setDateFilter();
setSelectFilter(array $values, $prompt);
setBooleanFilter();
```

Pokud je v Gridu potřeba sloupec z připojené tabulky a chceme v daném sloupci filtrovat záznamy, použije se metoda u sloupce setTableName(„table.column“).

```php
$dataSource = new NDataSource($articles->select("article.title, user.username AS username"));
...
$this->addColumn("username", "Uživatel")
        ->setTableName("user.username")
        ->setTextFilter();
```

Filtrování **textu**

|zadáno do inputu|filtruje
|---|---|
|text	|obsahuje  "text" |
|text%	|začíná na  "text"|
|%text	|končí na  "text" |

Filtrování **čísel** a **data**

|zadáno do inputu|filtruje
|---|---|
|1	|přesně 1|
|>1	|větší než 1|
|>=1|větší nebo rovno 1|
|<1	|menší než 1|
|<=1|menší nebo rovno 1|
|<>1|rozdílný od 1|


Autocomplete
------------

Autocomplete se nastavuje pro filtr a může být použit pouze s textovým filtrem. Nastavuje se až po filtru. Jediný parametr je počet vyhledávaných záznamů.

```php
$this->addColumn('column', 'Column')
    ->setTextFilter()
    ->setAutocomplete($numOfResults);
```

SubGridy
--------

Každý Grid může mit více SubGridů. Každý SubGrid může mít další SubGridy. Přidání SubGridy je velice jednoduché, viz kód:

```php
$this->addSubGrid("comments", "Zobrazit komentáře k článku")
    ->setGrid(new CommentGridByArticleId($presenter->context->database->table('comment'), $this->activeSubGridId))
    ->settings(function($grid){
        $grid->setWidth("800px");
        $grid['columns-title']->setWidth("400px");
    })
    ->setCellStyle("background-color:#f6f6f6; padding:20px;");
```

V metodě setGrid předáme jinou instanci Gridu, předem vytvořenou, například pro komentáře ke konkrétnímu článku. Druhý parametr Gridu je $this->activeSubGridId, což je id řádku, pro který chceme komentáře zobrazit a automaticky je při otevření SubGridu nastaven. V Gridu CommentGridByArticleId budeme omezovat výběr podmínkou na konkrétní id. Pokud chceme uplatnit nějaké nastavení po inicializaci SubGridu, můžeme použít metodu settings. Pro styl buňky subGridu použijeme metodu setCellStyle.

Takto by mohl vypadat jednoduchý Grid CommentGridByArticleId

```php
use \NiftyGrid\Grid;

class CommentGridByArticleId extends Grid
{
    protected $comments;

    protected $article_id;

    public function __construct($comments, $article_id)
    {
        parent::__construct();
        $this->comments = $comments;
        $this->article_id = $article_id;
    }

    protected function configure($presenter)
    {
        $source = new \NiftyGrid\NDataSource($this->comments->select('comment.id, title, user.username')->where('article_id = ?', $this->article_id));

        $this->setDataSource($source);

        $this->addColumn("title", "Titulek");
        $this->addColumn("username", "Autor");
    }

}
```