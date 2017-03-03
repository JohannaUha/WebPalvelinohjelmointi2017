Jatkamme sovelluksen rakentamista siitä, mihin jäimme viikon 6 lopussa. Allaoleva materiaali olettaa, että olet tehnyt kaikki edellisen viikon tehtävät. Jos et tehnyt kaikkia tehtäviä, voit ottaa kurssin repositorioista [edellisen viikon mallivastauksen](https://github.com/mluukkai/WebPalvelinohjelmointi2017/tree/master/mallivastaus/viikko6). Jos sait suurimman osan edellisen viikon tehtävistä tehtyä, saattaa olla helpointa, että täydennät vastaustasi mallivastauksen avulla.

Jos otat edellisen viikon mallivastauksen tämän viikon pohjaksi, kopioi hakemisto muualle kurssirepositorion alta (olettaen että olet kloonannut sen) ja tee sovelluksen sisältämästä hakemistosta uusi repositorio.

**Tehtävien deadline poikkeuksellisesti vasta maanantaina 6.3. klo 23.59**

## Muistutus debuggerista

Viikolla 2 tutustuimme [debuggeriin](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko2.md#debuggeri) ja [viime viikolla](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko6.md#muistutus-debuggerista) oli muistutus debuggerin käytösä. Edelleen edellisellä viikolla oli kuitenkin havaittavissa debuggerin alikäyttöä. Eli vielä kertauksena **kun kohtaat ongelman, turvaudu arvailun sijaan byebugiin!**

Rails-konsolin käytön tärkeyttä sovelluskehityksen välineenä on yritetty korostaa läpi kurssin. Eli **kun teet jotain vähänkin epätriviaalia, testaa asia ensin konsolissa.** Joissain tilanteissa voi olla jopa parempi tehdä kokeilut debuggerin avulla avautuvassa konsolissa, sillä tällöin on mahdollista avata konsolisessio juuri siihen kontekstiin, mihin koodia ollaan kirjoittamassa. Näin ollen päästään käsiksi esim. muuttujiin <code>params</code>, <code>sessions</code> ym. suorituskontekstista riippuvaan dataan.

## Testeistä

Osa tämän viikon tehtävistä saattaa hajottaa jotain edellisinä viikkoina tehtyjä testejä. Voit merkitä tehtävät testien hajoamisesta huolimatta, eli testien ja travisin pitäminen kunnossa on vapaaehtoista.

## Erilaiset järjestykset

Päätetään toteuttaa oluiden listalle toiminnallisuus, jonka avulla oluet voidaan järjestää eri sarakkeiden perusteella. Välitetään tieto halutusta järjestyksestä kontrollerille HTTP-pyynnön parametrina. Muutetaan näkymässä ```app/views/beers/index.html.erb``` olevaa taulukkoa seuraavasti:

```erb
<table class="table table-hover">
  <thead>
    <tr>
      <th> <%= link_to 'Name', beers_path(order:"name") %> </th>
      <th> <%= link_to 'Style', beers_path(order:"style") %> </th>
      <th> <%= link_to 'Brewery', beers_path(order:"brewery") %> </th>
    </tr>
  </thead>

  <tbody>
    <% @beers.each do |beer| %>
      <tr>
        <td><%= link_to beer.name, beer %></td>
        <td><%= link_to beer.style, beer.style %></td>
        <td><%= link_to beer.brewery.name, beer.brewery %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

eli taulukon sarakkeiden otsikoista on nyt tehty linkit, jotka johtavat takaisin samalle sivulle mutta lisäävät pyyntöön [query parametrin](https://en.wikipedia.org/wiki/Query_string) <code>:order</code>, joka määrittelee halutun järjestyksen. Käytännössä parametri välitetään urlin mukana liittäen se "normaalin" urlin perään kysymysmerkillä erottaen. Esim. jos klikataan tyylisaraketta, tulee urliksi _beers?order=style_

Kontrolleri pääsee HTTP-pyynnön parametriin käsiksi <code>params</code>-hashin avulla ja kuten olettaa saattaa, suunnan määrittelevän parametrin arvo on <code>params[:order]</code>.

Laajennetaan oluiden kontrolleria siten, että se testaa onko pyynnössä parametria, ja jos on, oluet järjestetään halutulla tavalla:

```ruby
  def index
    @beers = Beer.all

    order = params[:order] || 'name'

    @beers = case order
      when 'name' then @beers.sort_by{ |b| b.name }
      when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
      when 'style' then @beers.sort_by{ |b| b.style.name }
    end
  end
```

Koodi määrittelee järjestämisen tapahtuvan oletusarvoisesti nimen perusteella. Tämä tapahtuu seuraavasti

```ruby
    order = params[:order] || 'name'
```

Normaalisti <code>order</code> saa arvon <code>params[:order]</code>, jos parametria <code>:order</code> ei ole asetettu, eli sen arvo on <code>nil</code>, tulee arvoksi <code>||</code>:n jälkeinen osa eli 'name'.

**Huom1:** käytämme oluiden järjestämiseen rubyn <code>case when</code>-komentoa

```ruby
    @beers = case order
      when 'name' then @beers.sort_by{ |b| b.name }
      when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
      when 'style' then @beers.sort_by{ |b| b.style.name }
    end
```

joka toimii oleellisesti samoin kuin seuraava

```ruby
    @beers =
    if order == 'name'
      @beers.sort_by{ |b| b.name }
    elsif orded == 'brewery'
      @beers.sort_by{ |b| b.brewery.name }
    elsif orded == 'style'
      @beers.sort_by{ |b| b.style.name }
    end
```

**Huom2:** esimerkissä oluet haetaan ensin tietokannasta ja sen jälkeen järjestetään ne keskusmuistissa. Oluiden lista olisi mahdollista järjestää myös tietokantatasolla:

```ruby
   # oluet nimen perusteella järjestettynä
   Beer.order(:name)

   # oluet panimoiden nimien perusteella järjestettynä
   Beer.includes(:brewery).order("breweries.name")

   # oluet tyylin nimien perusteella järjestettynä
   Beer.includes(:style).order("style.name")
```

> ## Tehtävä 1
>
> Muuta panimot listaavaa sivua siten, että panimot voidaan järjestää nimen mukaiseen aakkosjärjestykseen tai perustamisvuoden mukaiseen järjestykseen. Nimen mukainen järjestys on oletusarvoinen. Viime viikolla laajensimme panimoiden listaa siten että aktiiviset ja lopettaneet panimot ovat omalla listallaan. Voit toteuttaa toiminnallisuuden siten, että molempien listojen järjestys on aina sama.

> ## Tehtävä 2
>
> **HUOM** Tämä tehtävä saattaa olla haastava saada toimimaan oikein. Älä jumita tähän tehtävään, seuraavat tehtävät eivät riipu tästä tehtävästä.
>
> Laajenna panimoiden järjestämistoimintoa siten, että jos panimot ovat esim. vuoden mukaan järjestettyjä eli saraketta _year_ on klikattu ja saraketta klikataan heti perään toistamiseen, järjestetään panimot vuoden mukaan käänteiseen järjestykseen.
>
> Vihje: joudut muistamaan panimoiden edellisen järjestyksen ja muistaminen taas onnistuu parhaiten [session](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko3.md#k%C3%A4ytt%C3%A4j%C3%A4-ja-sessio) avulla.
>
> Toteutuksesi saa vaihtaa järjestyssuuntaa myös siinä tapauksessa, että järjestysperuste vaihtuu esim. nimen perusteella tehtävästä vuoden perusteella tehtäväksi.

## Selainpuolella toteutettu toiminnallisuus

Ratkaisumme oluiden listan järjestämiseen on melko hyvä. Suorituskyvyn kannalta hieman ongelmallista on tosin se, että aina järjestettäessä tehdään kutsu palvelimelle, joka generoi uudessa järjestyksessä näytettävän sivun.

Järjestämistoiminnallisuus voitaisiin toteuttaa myös selaimen puolella javascriptillä. Vaikka kurssi keskittyy palvelinpuolen toiminnallisuuteen, näytetään seuraavassa esimerkki siitä, miten järjestämistoiminnallisuus toteutettaisiin selainpuolella. Tässä ratkaisussa palvelin tarjoaa ainoastaan oluiden listan json-muodossa, ja selaimessa suoritettava javascript-koodi hoitaa myös oluet listaavan taulukon muodostamisen.

Emme korvaa nyt olemassaolevaa oluiden listaa, eli sivun beers toiminnallisuutta, sen sijaan tehdään toiminnallisuutta varten kokonaan uusi, osoitteessa beerlist toimiva sivu. Tehdään sivua varten reitti tiedostoon routes.rb:

    get 'beerlist', to:'beers#list'

Käytämme siis olutkontrollerissa olevaa <code>list</code>-metodia. Metodin ei tarvitse tehdä mitään:

```ruby
class BeersController < ApplicationController
  before_action :ensure_that_signed_in, except: [:index, :show, :list]
  # muut before_actionit enallaan

  def list
  end

  ...
end
```

**HUOM** lisäsimme metodin <code>list</code> niiden joita ennen ei tarvitse suorittaa <code>ensure_that_signed_in</code>-metodia, eli oluiden javascriptilla tuotetun listan näkeminen ei edellytä sivulle kirjautumista!

Myös näkymä views/beers/list.html.erb on minimalistinen:

```erb
<h2>Beers</h2>

<div id="beers"></div>
```

Eli näkymä ainoastaan sijoittaa sivulle div-elementin, jolle annetaan id:ksi (eli viitteksi, jolla elementtiin päästään käsiksi) "beers".

Kuten odotettua, osoitteessa http://localhost:3000/beerlist ei nyt näy mitään muuta kuin h2-elementin sisältö.

Alamme nyt kirjoittamaan toimintalogiikan toteutusta javascriptillä hyödyntäen [JQuery](https://jquery.com/)-kirjastoa.

Jotta saamme JQueryn toimimaan hyvin yhteen Railsin ns. [turbolinks](https://github.com/rails/turbolinks)-ominaisuuden kanssa, otetaan käyttöön gemi https://github.com/kossnocorp/jquery.turbolinks eli lisätään Gemfileen rivi

    gem 'jquery-turbolinks'

tämän lisäksi tiedostoon app/assets/javascripts/application.js on **lisättävä** rivien

    //= require jquery
    //= require jquery_ujs

väliin rivi

    //= require jquery.turbolinks


Rails-sovelluksen tarvitsema javascript-koodi kannattaa sijoittaa hakemistoon app/assets/javascripts. Tehdään hakemistoon tiedosto _beerlist.js_ jolla on seuraava sisältö:

```javascript
$(document).ready(function () {
    $('#beers').html("hello from javascript");
    console.log("hello console!");
});
```

Kun sivu nyt avataan uudelleen, asetetaan javascriptillä tai tarkemmin sanottuna jQuery-kirjastolla id:n <code>beers</code> omaavaan elementtiin teksti "hello form javascript". Seuraava komento kirjoittaa javascript-konsoliin tervehdyksen.

Javascript-ohjelmoinnissa selaimessa oleva konsoli on **erittäin tärkeä** työväline. Konsolin saa avattua chromessa tools-valikosta tai painamalla ctrl, shift, j (linux) tai alt, cmd, i (mac):

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w7-1.png)

**Konsoli on syytä pitää koko ajan auki Javascriptillä ohjelmoitaessa!**

Javascript näyttää aluksi melko kryptiseltä, mm. paljon käytettyjen anonyymifunktioiden takia. Edellä oleva koodi määrittelee että <code>$(document).ready</code>-tapahtuman yhteydessä suoritetaan anonyymifunktion määrittelemä koodi:

```javascript
function () {
    $('#beers').html("hello from javascript");
    console.log("hello console!");
}
```

<code>$(document).ready</code> taas on tapahtuma (engl. event), joka tapahtuu koko www-sivun HTML-koodin ollessa latautunut selaimelle.


Jos kokeilemme selaimella osoitetta [http://localhost:3000/beers.json](http://localhost:3000/beers.json) huomaamme, että saamme vastaukseksi oluiden tiedot tekstuaalisessa json-muodossa (ks. http://en.wikipedia.org/wiki/JSON, http:www.json.org):

```ruby
[{"id":6,"name":"Hefeweizen","style":{"id":4,"name":"Weizen","description":"A south German style of wheat beer (weissbier) made with a typical ratio of 50:50, or even higher, wheat. A yeast that produces a unique phenolic flavors of banana and cloves with an often dry and tart edge, some spiciness, bubblegum or notes of apples. Little hop bitterness, and a moderate level of alcohol. The \"Hefe\" prefix means \"with yeast\", hence the beers unfiltered and cloudy appearance. Poured into a traditional Weizen glass, the Hefeweizen can be one sexy looking beer. \r\n\r\nOften served with a lemon wedge (popularized by Americans), to either cut the wheat or yeast edge, which many either find to be a flavorful snap ... or an insult and something that damages the beer's taste and head retention.","created_at":"2017-02-05T17:44:15.892Z","updated_at":"2017-02-05T18:25:02.337Z"},"brewery_id":3,"url":"http://localhost:3000/beers/6.json"},{"id":7,"name":"Helles","style":{"id":1,"name":"European pale lager","description":"Similar to the Munich Helles story, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweetish notes from an all-malt base.","created_at":"2017-02-05T17:44:15.873Z","updated_at":"2017-02-05T18:21:13.441Z"},"brewery_id":3,"url":"http://localhost:3000/beers/7.json"},{"id":4,"name":"Huvila Pale Ale","style":{"id":2,"name":"American pale ale","description":"Of British origin, this style is now popular worldwide and the use of local ingredients, or imported, produces variances in character from region to region. Generally, expect a good balance of malt and hops. Fruity esters and diacetyl can vary from none to moderate, and bitterness can range from lightly floral to pungent. \r\n\r\nAmerican versions tend to be cleaner and hoppier, while British tend to be more malty, buttery, aromatic and balanced.","created_at":"2017-02-05T17:44:15.887Z","updated_at":"2017-02-05T18:22:25.674Z"},"brewery_id":2,"url":"http://localhost:3000/beers/4.json"},{"id":1,"name":"Iso 3","style":{"id":1,"name":"European pale lager","description":"Similar to the Munich Helles story, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweetish notes from an all-malt base.","created_at":"2017-02-05T17:44:15.873Z","updated_at":"2017-02-05T18:21:13.441Z"},"brewery_id":1,"url":"http://localhost:3000/beers/1.json"},{"id":2,"name":"Karhu","style":{"id":1,"name":"European pale lager","description":"Similar to the Munich Helles story, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweetish notes from an all-malt base.","created_at":"2017-02-05T17:44:15.873Z","updated_at":"2017-02-05T18:21:13.441Z"},"brewery_id":1,"url":"http://localhost:3000/beers/2.json"}]
```

Json-muotoisen sivun saa hieman luettavampaan muotoon esim. kopioimalla sivun sisällön [jsonlint](http://jsonlint.com/) palveluun:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w7-3.png)

Parempi ratkaisu on asentaa selaimeen jsonia ymmärtävä plugin, eräs suositeltava on chromen [jsonview](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc), plugin muotoilee jsonin selaimeen todella siististi:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w7-4.png)

Tarkemmin tarkasteltuna jokainen yksittäinen json-muotoinen olut muistuttaa hyvin paljon Rubyn hashiä:

```ruby
{"id":6,"name":"Hefeweizen","style":{"id":4,"name":"Weizen","description":"A south German style of wheat beer (weissbier) made with a typical ratio of 50:50, or even higher, wheat. A yeast that produces a unique phenolic flavors of banana and cloves with an often dry and tart edge, some spiciness, bubblegum or notes of apples. Little hop bitterness, and a moderate level of alcohol. The \"Hefe\" prefix means \"with yeast\", hence the beers unfiltered and cloudy appearance. Poured into a traditional Weizen glass, the Hefeweizen can be one sexy looking beer. \r\n\r\nOften served with a lemon wedge (popularized by Americans), to either cut the wheat or yeast edge, which many either find to be a flavorful snap ... or an insult and something that damages the beer's taste and head retention.","created_at":"2017-02-05T17:44:15.892Z","updated_at":"2017-02-05T18:25:02.337Z"},"brewery_id":3,"url":"http://localhost:3000/beers/6.json"}
```

Minkä takia Rails osaa tarvittaessa palauttaa resurssit HTML:n sijaan jsonina?

Yritetään saada kaikkien reittausten lista jsonina, eli kokeillaan osoitetta [http://localhost:3000/ratings.json](http://localhost:3000/ratings.json)

Seurauksena on virheilmoitus:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-1.png)

Eli ihan automaattisesti jsonit eivät synny, loimme kaiken reittaukseen liittyvän koodin käsin, ja kuten virheilmoituksesta voimme päätellä, formaatille 'json' ei ole olemassa sopivaa templatea.

Huomaamme, että scaffoldilla luotujen resurssien, esim oluen views-hakemistosta löytyy joukko _json.jbuilder_-päätteisiä templateja, ja kuten arvata saattaa, käyttää Rails näitä jos resurssi halutaan json-muotoisena.

Ottamalla mallia templatesta app/views/beers/index.json.jbuilder teemme reittauksille seuraavan json.jbuilder-templaten (tiedosto on siis app/views/ratings/index.json.jbuilder):

```ruby
json.array!(@ratings) do |rating|
  json.extract! rating, :id, :score
end
```

ja nyt saamme reittaukset jsonina osoitteesta [http://localhost:3000/ratings.json](http://localhost:3000/ratings.json)

```ruby
[{"id":1,"score":13},{"id":2,"score":34},{"id":3,"score":25}]
```

*HUOM:* jbuilder-templatessa käytettävän muuttujan <code>@ratings</code>
 tulee olla määritelty kontrollerin metodissa <code>index</code>!

Voisimme helposti määritellä json.jbuilder-templatessa, että reittausten json-esitykseen sisällytetään myös reittausta koskevan oluen tiedot:

```ruby
json.array!(@ratings) do |rating|
  json.extract! rating, :id, :score, :beer
end
```

Lisää jbuilderista seuraavissa http://railscasts.com/episodes/320-jbuilder?autoplay=true ja https://github.com/rails/jbuilder

Json-jbuilder-templatejen ohella toinen tapa palauttaa json-muotoista dataa olisi käytää <code>respond_to</code>-komentoa, jota muutamat scaffoldienkin generoivat metodit käyttävät. Tällöin json-jbuilder-templatea ei tarvittaisi ja kontrolleri näyttäisi seuraavalta

```ruby
  def index
    @ratings = Rating.all

    respond_to do |format|
      format.html { } # renderöidään oletusarvoinen template
      format.json { render json: @ratings }
    end
  end
```

Jbuilder-templatejen käyttö on kuitenkin ehdottomasti parempi vaihtoehto, tällöin json-muotoisen "näytön" eli resurssin representaation muodostaminen eriytetään täysin kontrollerista. Ei ole kontrollerin vastuulla muotoilla vastauksen ulkoasua oli kyseessä sitten json- tai HTML-muotoinen vastaus.

Palataan oluiden sivun pariin. Kun muodostamme sivun javascriptillä, ideana onkin hakea palvelimelta nimenomaan oluet json-muodossa ja renderöidä ne sitten sopivasti javascriptin avulla.

Muokataan javascript-koodiamme seuraavasti:

```javascript
$(document).ready(function () {
    $.getJSON('beers.json', function (beers) {
        oluet = beers
        $("#beers").html("oluita löytyi "+beers.length);
    });
});
```

Koodin ensimmäinen rivi (joka siis suoritetaan heti kun sivu on latautunut) tekee HTTP GET -kyselyn palvelimen osoitteeseen beers.json ja määrittelee takaisinkutsufunktion, jota selain kutsuu siinä vaiheessa kun GET-kyselyyn tulee vastaus palvelimelta. Takaisinkutsufunktion parametrissa <code>beers</code> on palvelimelta tullut data, eli json-muodossa oleva oluiden lista. Muuttujan sisältö sijoitetaan globaaliin muuttujaan <code>oluet</code> ja oluiden listan pituus näytetään www-sivulla id:n beers omaavassa elementissä.

Koska sijoitimme viitteen oluiden listan globaaliin muuttujaan, voimme tarkastella sitä selaimen konsolista käsin (muistutuksena että konsolin saa avattua chromessa tools-valikosta tai painamalla ctrl, shift, j (linux) tai alt, cmd, i (mac) ja jos olit jo sulkenut konsolin teit pahan virheen. **Konsoli tulee pitää aina auki javascriptillä ohjelmoitaessa!**):

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w7-2.png)

Takaisinkutsufunktion pitäisi siis saatuaan oluet palvelimelta muodostaa ne listaava HTML-koodi ja lisätä se sivulle.

Muutetaan javascript-koodiamme siten, että se listaa aluksi ainoastaan oluiden nimet:

```javascript
    $.getJSON('beers.json', function (beers) {
        var beer_list = [];

        $.each(beers, function (index, beer) {
            beer_list.push('<li>' + beer['name'] + '</li>')
        });

        $("#beers").html('<ul>'+ beer_list.join('') + '</ul>');
     });
```

Koodi määrittelee paikallisen taulukkomuuttujan <code>beer_list</code> ja käy läpi parametrina saamansa oluiden listan <code>beers</code> (hieman erikoiselta näyttävää jqueryn <code>each</code>-metodia käyttäen, ks. http://api.jquery.com/jQuery.each/), ja lisää jokaista olutta kohti <code>beer_list</code>:iin HTML-elementin, joka on muotoa

```erb
    <li>Karhu tuplahumala</li>
```

Lopuksi listan alkuun ja loppuun lisätään ul-tagit ja listan alkiot liitetään yhteen join-metodilla. Näin saatu HTML-koodi liitetään id:n <code>beers</code> omaavaan elementtiin.

Nyt siis saimme yksinkertaisen listan oluiden nimistä sivulle.

Entä jos haluaisimme järjestää oluet? Jotta tämä onnistuu, refaktoroimme koodin ensin seuraavanlaiseksi:

```javascript
var BEERS = {};

BEERS.show = function(){
    var beer_list = [];

    $.each(BEERS.list, function (index, beer) {
        beer_list.push('<li>' + beer['name'] + '</li>')
    });

    $("#beers").html('<ul>'+ beer_list.join('') + '</ul>');
};

$(document).ready(function () {

    $.getJSON('beers.json', function (beers) {
        BEERS.list = beers;
        BEERS.show();
     });
});
```

Määrittelimme nyt olion <code>BEERS</code>, jonka attribuuttiin <code>BEERS.list</code> palvelimelta saapuva oluiden lista sijoitetaan. Metodi <code>BEERS.show</code> muodostaa <code>BEERS.list</code>:in oluista HTML-taulukon ja sijoittaa sen näytölle.

Näin muotoiltuna palvelimelta haettu oluiden lista jää "muistiin" selaimeen muuttujaan <code>BEERS.list</code> ja lista voidaan tarpeen tullen uudelleenjärjestää ja näyttää käyttäjälle uudessa järjestyksessä ilman että www-sivun tarvitsee ollenkaan kommunikoida palvelimen kanssa.

Lisätään sivulle painike (tai linkki), jota painamalla oluet saadaan sivulle käänteiseen järjestykseen:

```erb
<a href="#" id="reverse">reverse!</a>
<div id="beers"></div>
```

Lisätään sitten javascriptillä linkille klikkauksenkäsittelijä, joka linkkiä klikatessa laittaa oluet käänteiseen järjestykseen ja näyttää ne sivun beers-elementissä:

```javascript
var BEERS = {};

BEERS.show = function(){
    var beer_list = [];

    $.each(BEERS.list, function (index, beer) {
        beer_list.push('<li>' + beer['name'] + '</li>')
    });

    $("#beers").html('<ul>'+ beer_list.join('') + '</ul>');
};

BEERS.reverse = function(){
    BEERS.list.reverse();
};

$(document).ready(function () {
    $("#reverse").click(function (e) {
        BEERS.reverse();
        BEERS.show();
        e.preventDefault();
    });

    $.getJSON('beers.json', function (beers) {
        BEERS.list = beers;
        BEERS.show();
    });
});
```

Linkin klikkauksen käsittelijä siis määritellään tapahtuman <code>document ready</code> sisällä, eli kun dokumentti on latautunut, _rekisteröidään_ klikkausten käsittelijäfunktio id:n "reverse" omaavalle linkkielementille. Selain kutsuu määriteltyä funktiota kun linkkiä klikataan. Viimeiseksi komento <code>preventDefault</code> estää klikkauksen "normaalin" toiminnallisuuden eli (nyt olemattoman) linkin seuraamisen.

Nyt ymmärrämme riittävästi perusteita ja olemme valmiina toteuttamaan todellisen toiminnallisuuden.

Muutetaan näkymää seuraavasti:

```erb
<h2>Beers</h2>

<table id="beertable" class="table table-hover">
  <tr>
    <th> <a href="#" id="name">Name</a> </th>
    <th> <a href="#" id="style">Style</a> </th>
    <th> <a href="#" id="brewery">Brewery</a> </th>
  </tr>

</table>
```

Eli kolmesta sarakenimestä on tehty linkki, joihin tullaan rekisteröimään klikkauksenkuuntelijat. Taulukolle on annettu id <code>beertable</code>.

Muutetaan sitten javascriptissä määriteltyä metodia <code>show</code> siten, että se laittaa oluiden nimet taulukkoon:

```javascript
BEERS.show = function(){
    var table = $("#beertable");

    $.each(BEERS.list, function (index, beer) {
        table.append('<tr><td>'+beer['name']+'</td></tr>');
    });
};
```

Eli ensin koodi tallettaa viitteen taulukkoon muuttujana <code>table</code> ja lisää sinne <code>append</code>-komennolla uuden rivin kutakin olutta varten.

Laajennetaan sitten metodia näyttämään kaikki tiedot oluista. Huomaamme kuitenkin, että oluiden json-muotoisessa listassa <http://localhost:3000/beers.json> ei ole panimosta muuta tietoa kuin olioiden id:t, haluaisimme kuitenkin näyttää panimon nimen. Oluttyylin tiedot löytyvät kokonaisuudessaan jsonista jo nyt.

Ongelma on onneksi helppo ratkaista muokkaamalla oluiden listan tuottavaa json-jbuildertemplatea. Template näyttää nyt seuraavalta:

```ruby
json.array!(@beers) do |beer|
  json.extract! beer, :id, :name, :style, :brewery_id
  json.url beer_url(beer, format: :json)
end
```

Template määrittelee, että jokaisesta oluesta json-esitykseen sisällytetään kentät _id_, _name_ ja _brewery_id_ sekä _style_ joka taas viittaa olueeseen liittyvään <code>Style</code>-olioon. Tyyliolio tuleekin renderöityä oluen json-esityksen sisälle kokonaisuudessaan. Saamme myös panimon json-esityksen oluen jsonin mukaan jos korvaamme templatessa _brewery_id_:n _brewery_:llä. Muutamme siis templaten muotoon:

```ruby
json.array!(@beers) do |beer|
  json.extract! beer, :id, :name, :style, :brewery
end
```

poistimme viimeisen rivin joka lisäsi jokaisen oluen json-esityksen mukaan urlin oluen omaan json-esitykseen.

Nyt saamme taulukon generoitua seuraavalla javascriptillä:

```javascript
BEERS.show = function(){
    var table = $("#beertable");

    $.each(BEERS.list, function (index, beer) {
        table.append('<tr>'
                        +'<td>'+beer['name']+'</td>'
                        +'<td>'+beer['style']['name']+'</td>'
                        +'<td>'+beer['brewery']['name']+'</td>'
                    +'</tr>');
    });
};
```

Oluiden listan json-esityksen mukana tulee nyt paljon tarpeetontakin tietoa sillä mukaan renderöityvät jokaisen oluen panimon ja tyylin json-esitykset kokonaisuudessaan. Voisimme optimoida templatea siten, että oluen panimosta ja tyylistä tulee json-esitykseen mukaan ainoastaan nimi:

```ruby
json.array!(@beers) do |beer|
  json.extract! beer, :id, :name
  json.style do
    json.name beer.style.name
  end
  json.brewery do
    json.name beer.brewery.name
  end
end
```

Nyt palvelimen lähettämä oluiden jsonmuotoinen lista on huomattavasti inhimillisemmän kokoinen:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w7-5.png)

Rekisteröimme vielä järjestämisen suorittavat tapahtumankuuntelijat linkeille (seuraavassa lopullinen javascript-koodi):

```javascript
var BEERS = {};

BEERS.show = function(){
    $("#beertable tr:gt(0)").remove();

    var table = $("#beertable");

    $.each(BEERS.list, function (index, beer) {
        table.append('<tr>'
            +'<td>'+beer['name']+'</td>'
            +'<td>'+beer['style']['name']+'</td>'
            +'<td>'+beer['brewery']['name']+'</td>'
            +'</tr>');
    });
};

BEERS.sort_by_name = function(){
    BEERS.list.sort( function(a,b){
        return a.name.toUpperCase().localeCompare(b.name.toUpperCase());
    });
};

BEERS.sort_by_style = function(){
    BEERS.list.sort( function(a,b){
        return a.style.name.toUpperCase().localeCompare(b.style.name.toUpperCase());
    });
};

BEERS.sort_by_brewery = function(){
    BEERS.list.sort( function(a,b){
        return a.brewery.name.toUpperCase().localeCompare(b.brewery.name.toUpperCase());
    });
};

$(document).ready(function () {
    $("#name").click(function (e) {
        BEERS.sort_by_name();
        BEERS.show();
        e.preventDefault();
    });

    $("#style").click(function (e) {
        BEERS.sort_by_style();
        BEERS.show();
        e.preventDefault();
    });

    $("#brewery").click(function (e) {
        BEERS.sort_by_brewery();
        BEERS.show();
        e.preventDefault();
    });

    $.getJSON('beers.json', function (beers) {
        BEERS.list = beers;
        BEERS.sort_by_name();
        BEERS.show();
    });

});
```

Javascript-koodimme tulee liitetyksi sovelluksen jokaiselle sivulle. Tästä on se ikävä seuraus, että ollaanpa millä sivulla tahansa, lataa javascript oluiden listan komennon <code>getJSON('beers.json', ...) </code> takia. Myös tapahtumakunntelijat yritetään rekisteröidä jokaiselle sivulle vaikka niiden rekisteröinti on mielekästä ainoastaan jos ollaan oluiden listalla.

Viritellään javascript-koodia vielä siten, että <code>$(document).ready</code>:n sisällä oleva koodi suoritetaan ainoastaan jos ollaan sivulla, josta taulukko <code>beertable</code> löytyy:

```javascript
$(document).ready(function () {
    if ( $("#beertable").length>0 ) {

      $("#name").click(function (e) {
          BEERS.sort_by_name();
          BEERS.show();
          e.preventDefault();
      });

      $("#style").click(function (e) {
          BEERS.sort_by_style();
          BEERS.show();
          e.preventDefault();
      });

      $("#brewery").click(function (e) {
          BEERS.sort_by_brewery();
          BEERS.show();
          e.preventDefault();
      });


      $.getJSON('beers.json', function (beers) {
        BEERS.list = beers;
        BEERS.sort_by_name;
        BEERS.show();
      });

    }
});
```

Tällä hetkellä trendinä siirtää yhä suurempi osa web-sivujen toiminnallisuudesta selaimeen. Etuna mm. se että web-sovelluksien toiminta saadaan muistuttamaan yhä enenevissä määrin desktop-sovelluksia.

## AngularJS ja React 

Äsken javascriptillä toteuttamamme oluet listaava sivu oli koodin rakenteen puolesta ihan kohtuullista, mutta Railsin sujuvuuteen ja vaivattomuuteen verrattuna koodi oli raskaahkoa ja paikoin ikävien, rutiininomaisten yksityiskohtien täyttämää. Jos sovelluksen selainpuolella toteutettavan koodin määrä alkaa kasvaa, on lopputuloksena helposti sekava koodi, jonka toiminnasta kukaan ei enää ota selvää ja jonka laajentaminen muuttuu erittäin haastavaksi.

Javascript-frontendsovelluskehykset tuovat asiaan helpotusta. Viime vuoden [materiaalissa](https://github.com/mluukkai/WebPalvelinohjelmointi2016/blob/master/web/viikko7.md#angularjs) tutustuimme nopeasti [AngularJS](https://angularjs.org)-kirjastoon. AngularJS oli hetken maailman ylivoimaisesti suosituin fronendsovelluskehys, mutta version [2.0](https://angular.io) ilmestymisen jälkeen se menetti nopeasti asemansa [React](https://facebook.github.io/react/)-nimiselle kirjastolle. 

[React](https://facebook.github.io/react/) on tällä hetkellä hypekäyrän huipuilla. Reactin käyttö on kuitenkin siinä määrin kompleksista että emme tällä kurssilla voi tutustua siihen edes pintapuolisesti.

Jos olet kiinnostunut Reactista ja haluat käyttää sitä esim. periodin 4 [harjoitustyössä](https://github.com/mluukkai/WebPalvelinohjelmointi2017/wiki/projekti), voit tutustua aiheeseen osoitteesta <https://github.com/mluukkai/reactbeer> löytyvässä pienessä tutoriaalissa. Reactbeerissä käydään läpi myös muutamia asioita, joita tulee huomioida siinä tapauksessa että Railsia käytetään ensisijaisesti _json-muotoista_ dataa tarjoavana "backendinä".

Javascript-maailma on tällä hetkellä erittäin turbulentissa ja monia turhauttavassa tilassa (ks. esim. [Javascript fatigue](http://thefullstack.xyz/javascript-fatigue/>) ja [How it feels to learn Javascript](https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f#.vg5ddumd2)). React näyttää juuri nyt siltä hevoselta minkä varaan rahat kannattaa pelata. Vuoden tai parin päästä tilanne voi jo olla ihan toinen. Juuri tällä hetkellä eniten js-hipsterien silmissä asemiaan nostava frontendsovelluskehys on <https://vuejs.org>. 

> ## Tehtävä 3
>
> Toteuta edellisten esimerkkien tyyliin javascriptillä  kaikki panimot listaava sivu http:localhost:3000/brewerylist  
> Sivulla näytetään jokaisesta panimosta nimi, perustusvuosi, panimon valmistamien oluiden lukumäärä ja tieto siitä onko panimo lopettanut. Sivun siis **ei** tarvitse eritellä lopettaneita panimoita omaan taulukkoonsa.
> Panimoiden järjestäminen toteutetaan vasta seuraavassa tehtäässä.
>
> **Muista pitää Javascript-konsoli koko ajan auki tehtävää tehdessäsi!** Voit debugata Javasriptia tulostelemalla konsoliin komennolla <code>console.log()</code>
>
> **HUOM:** edellisellä viikolla tekmämme muutoksen takia panimoiden json-lista http://localhost:3000/breweries.json ei toimi, sillä breweries#index-kontrolleri ei enää aseta kaikkien panimoiden listaa muuttujaan <code>@breweries</code>. Korjaa tilanne.
>
> **HUOM2:** tehtävä kannattaa tehdä yksi pieni askel kerrallaan, samaan tapaan kuin oluiden lista tehtiin yllä olevassa esimerkissä. Javascriptin debuggaus saattaa olla haasteellista ja **varmin tapa aiheuttaa iso turhautuma onkin yrittää tehdä tehtävä nopeasti copypasteamalla beerlistin koodi**.

> ## Tehtävä 4
>
> Laajenna panimoiden listaa siten, että panimot voi järjestää joko aakkos- tai perustamisvuoden mukaiseen järjestykseen tai panimon valmistamien oluiden lukumäärän perusteella. 

## Selainpuolella toteutetun toiminnallisuuden testaaminen

Tehdään rspec/capybaralla muutama testi javascriptillä toteutetulle oluiden listalle. Seuraavassa on lähtökohtamme, tiedosto spec/features/beerlist_page_spec.rb:

```ruby
require 'rails_helper'

describe "Beerlist page" do
  before :each do
    @brewery1 = FactoryGirl.create(:brewery, name:"Koff")
    @brewery2 = FactoryGirl.create(:brewery, name:"Schlenkerla")
    @brewery3 = FactoryGirl.create(:brewery, name:"Ayinger")
    @style1 = Style.create name:"Lager"
    @style2 = Style.create name:"Rauchbier"
    @style3 = Style.create name:"Weizen"
    @beer1 = FactoryGirl.create(:beer, name:"Nikolai", brewery: @brewery1, style:@style1)
    @beer2 = FactoryGirl.create(:beer, name:"Fastenbier", brewery:@brewery2, style:@style2)
    @beer3 = FactoryGirl.create(:beer, name:"Lechte Weisse", brewery:@brewery3, style:@style3)
  end

  it "shows one known beer" do
    visit beerlist_path
    expect(page).to have_content "Nikolai"
  end
end
```

Suoritetaan testi komennolla <code>rspec spec/features/beerlist_page_spec.rb</code>. Tuloksena on kuitenkin virheilmoitus:

```ruby
  1) beerlist page Beerlist page shows one known beer
     Failure/Error: expect(page).to have_content "Nikolai"
       expected to find text "Nikolai" in "breweries beers styles ratings users clubs places | signin signup Beers Name Style Brewery"
```

Näyttää siis siltä että sivulla ei ole ollenkaan oluiden listaa. Varmistetaan tämä laittamalla testiin juuri ennen komentoa <code>expect</code> komento <code>save_and_open_page</code> jonka avulla saamme siis avattua selaimeen sivun jolle capybara on navigoinut
(ks. https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko4.md#capybarav4#capybara).

Ja aivan kuten arvelimme, sivulla näytettävä oluttaulukko on tyhjä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-2.png)

Syy ongelmalle löytyy capybaran dokumentaatiosta https://github.com/jnicklas/capybara#drivers

> By default, Capybara uses the :rack_test driver, which is fast but limited: it does not support JavaScript, nor is it able to access HTTP resources outside of your Rack application, such as remote APIs and OAuth services. To get around these limitations, you can set up a different default driver for your features.

Ja korjauskin on helppo. Javascriptiä tarvitseviin testeihin riittää lisätä parametri, jonka ansiosta testi suoritetaan javascriptiä osaavan Selenium-testiajurin avulla:

```ruby
    it "shows the known beers", js:true do
```

Jotta selenium saadaan käyttöön, on Gemfilen test-scopeen lisättävä seuraava gem:

    gem 'selenium-webdriver', '2.53.4'

Suoritetaan <code>bundle install</code>, ja ajetaan testit. Jälleen törmäämme virheilmoitukseen:

```ruby
     Failure/Error: visit beerlist_path
     WebMock::NetConnectNotAllowedError:
       Real HTTP connections are disabled. Unregistered request: GET http://127.0.0.1:60873/__identify__ with headers {'Accept'=>'*/*', 'Accept-Encoding'=>'gzip;q=1.0,deflate;q=0.6,identity;q=0.3', 'User-Agent'=>'Ruby'}
```

Virheen syy on siinä, että otimme viikolla 5 käyttöömme WebMock-gemin joka oletusarvoisesti kieltää testikoodin suorittamat HTTP-yhteydet. Javascriptilla toteutettu olutlistahan yrittää hakea oluiden listan json-muodossa palvelimelta. Pääsemme virheestä eroon sallimalla yhteydet paikalliselle palvelimelle, eli lisäämällä esim.  <code>before</code>-lohkon alkuun seuraavan komennon:

    WebMock.disable_net_connect!(allow_localhost:true)

Testi toimii vihdoin, mutta ei mene läpi. Huomaamme että korjausta alkuperäiseen ongelmaan ei tapahdu: vaikka <code>before :each</code> -lohkossa lisäämme oluita tietokantaan, vaikuttaa tietokanta tyhjältä.

Syynä tälle on se, että ajettaessa testejä Seleniumin kautta, rspecin normaali tapa suorittaa testi yhtenä tietokantatransaktiona (joka automaattisesti suorittaa rollback-operaation testin lopussa nollaten näin tietokantaan tapahtuneet muutokset) ei ole tuettu (ks. https://github.com/jnicklas/capybara#transactions-and-database-setup). Joudummekin kytkemään ominaisuuden pois päältä Seleniumin kautta ajettavien testien aluksi komennolla <code>self.use_transactional_fixtures = false</code>

Ikävä seuraus tästä on se, että testien tietokantaan tallettama data ei nyt automaattisesti poistu jokaisen testin jälkeen. DatabaseCleaner gemi https://github.com/bmabey/database_cleaner tuo kuitenkin avun tähän. Otetaan gem käyttöön määrittelemällä Gemfilen test-scopeen seuraava

    gem 'database_cleaner'

Suoritetaan sitten tuttu <code>bundle install</code>.

Konfiguroidaan testit siten, että _ennen kaikkia testejä_ (<code>before :all</code>) laitetaan transaktionaalisuus pois päältä, sallitaan HTTP-yhteydet paikalliselle palvelimelle ja määritellään DatabaseCleanerille käytettävä strategia (ks. http://stackoverflow.com/questions/10904996/difference-between-truncation-transaction-and-deletion-database-strategies). _Jokaisen testin alussa_ (<code>before :each</code>) käynnistetään DatabaseCleaner ja jokaisen testin lopussa (<code>after :each</code>) pyydetään DatabaseCleaneria tyhjentämään tietokanta. Kun kaikki testit on suoritettu (<code>after :all</code>) palautetaan normaali transaktionaalisuus:

```ruby
require 'rails_helper'

describe "beerlist page" do

  before :all do
    self.use_transactional_fixtures = false
    WebMock.disable_net_connect!(allow_localhost:true)
  end

  before :each do
    DatabaseCleaner.strategy = :truncation
    DatabaseCleaner.start

    @brewery1 = FactoryGirl.create(:brewery, name: "Koff")
    @brewery2 = FactoryGirl.create(:brewery, name: "Schlenkerla")
    @brewery3 = FactoryGirl.create(:brewery, name: "Ayinger")
    @style1 = Style.create name: "Lager"
    @style2 = Style.create name: "Rauchbier"
    @style3 = Style.create name: "Weizen"
    @beer1 = FactoryGirl.create(:beer, name: "Nikolai", brewery: @brewery1, style: @style1)
    @beer2 = FactoryGirl.create(:beer, name: "Fastenbier", brewery: @brewery2, style: @style2)
    @beer3 = FactoryGirl.create(:beer, name: "Lechte Weisse", brewery: @brewery3, style: @style3)
  end

  after :each do
    DatabaseCleaner.clean
  end

  after :all do
    self.use_transactional_fixtures = true
  end

  it "shows one known beer", js: true do
    visit beerlist_path
    save_and_open_page
    expect(page).to have_content "Nikolai"
  end
end
```

Jouduimme jo näkemään hieman vaivaa, mutta testi toimii vihdoin!

Kun sivuille luodaan sisältöä javascriptillä, ei sisältö ilmesty sivulle vielä samalla hetkellä kuin sivun html-pohja ladataan vaan vasta javascript takaisinkutsufunktion suorituksen jälkeen. Eli jos katsomme sivun sisältöä välittömästi sivulle navigoinnin jälkeen, ei javascript ole vielä ehtinyt muodostaa sivun lopullista sisältöä. Esim. seuraavassa <code>save_and_open_page</code> saattaa avata sivun, jossa ei vielä näy yhtään olutta:

``` ruby
  it "shows a known beer", js:true do
    visit beerlist_path
    save_and_open_page
    expect(page).to have_content "Nikolai"
  end
```

Kuten sivulla https://github.com/jnicklas/capybara#asynchronous-javascript-ajax-and-friends sanotaan, osaa capybara odottaa asynkroonisia javascript-kutsuja sen verran, että testien sivulta etsimät elementit ovat latautuneet.

Tiedämme, että javascriptin pitäisi lisätä sivun taulukkoon rivejä. Saammekin sivun näkymään oikein, jos lisäämme alkuun komennon <code>find('table').find('tr:nth-child(2)')</code> joka etsii sivulta taulukon ja sen sisältä toisen rivin (taulukon ensimmäinen rivihän on jo sivupohjassa mukana oleva taulukon otsikkorivi):

``` ruby
  it "shows a known beer", :js => true do
    visit beerlist_path
    find('table').find('tr:nth-child(2)')
    save_and_open_page
    expect(page).to have_content "Nikolai"
  end
```

Nyt capybara odottaa taulukon valmistumista ja siirtyy sivun avaavaan komentoon vasta taulukon latauduttua (itseasiassa vain 2 riviä taulukkoa on varmuudella valmiina).

> ## Tehtävä 5
>
> Tee testi joka varmistaa, että oluet ovat beerlist-sivulla oletusarvoisesti nimen mukaan aakkosjärjestyksessä
>
> Testaaminen kannattaa tehdä nyt siten, että etsitään taulukon rivit <code>find</code>-selektorin avulla ja varmistetaan, että jokaisella rivillä on oikea sisältö. Koska taulukossa on otsikkorivi, löytyy ensimmäinen varsinainen rivi seuraavasti:
>
> ``` ruby
> find('table').find('tr:nth-child(2)')
> ``` 
>
> Rivin sisältöä voi testata normaaliin tapaan expect ja have_content -metodeilla.

> ## Tehtävä 6
>
> Tee testit seuraaville toiminnallisuuksille
> * klikattaessa saraketta 'style' järjestyvät oluet tyylin nimen mukaiseen aakkosjärjestykseen
> * klikattaessa saraketta 'brewery' järjestyvät oluet panimon nimen mukaiseen aakkosjärjestykseen

**Huom.** Travis ei osaa suoraan ajaa Selenium-testejä. Ongelmaan löytyy vastaus täältä  http://about.travis-ci.org/docs/user/gui-and-headless-browsers/#Using-xvfb-to-Run-Tests-That-Require-GUI-(e.g.-a-Web-browser)
Travisin toimintaansaattaminen muutosten jälkeen on vapaaehtoista.

## Asset pipeline

Rails-sovelluksiin liittyviä javascript- ja tyylitiedostoja (ja kuvia) hallitaan ns. Asset pipelinen avulla, ks. http://guides.rubyonrails.org/asset_pipeline.html

Periaatteena on se, että sovelluskehittäjä sijoittaa sovellukseen liittyvät javascript-tiedostot hakemistoon _app/assets/javascripts_ ja tyylitiedostot hakemistoon _app/assets/stylesheets_.  Molempia voidaan sijoittaa useaan eri tiedostoon, ja tarvittaessa alihakemistoihin.

Sovellusta kehitettäessä (eli kun sovellus on ns. development-moodissa) Rails liittää kaikki (ns. manifest-tiedostossa) määritellyt javascript- ja tyylitiedostot mukaan sovellukseen. Huomaammekin tarkastellessamme sovellusta selaimen view source -ominaisuuden avulla, että mukaan on liitetty suuri joukko javascriptiä ja tyylitiedostoja.

Sovelluksen mukaan liitettävät javascript-tiedostot määritellään tiedostossa _app/assets/javascripts/application.js_, jonka sisältö on nyt seuraava

```javascript
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require bootstrap
//= require_tree .
```

Vaikka koko tiedoston sisältö näyttää olevan kommenteissa, on kuitenkin kyse "oikeista", asset pipelinestä huolehtivan [sprockets-kääntäjän](https://github.com/sstephenson/sprockets) komennoista, joiden avulla määritellään sovellukseen mukaan otettavat javascript-tiedostot. Neljä ensimmäistä riviä määrittelevät, että mukaan otetaan jquery, jquery_ujs, turbolinks ja bootstrap. Kaikki näistä on asennettu sovellukseen gemien avulla.

Viimeinen rivi määrittelee, että kaikki hakemiston *assets/javascripts/* ja sen alihakemistojen sisältämät javascript-tiedostot sisällytetään ohjelmaan.

Asset pipeline mahdollistaa myös [coffeescriptin](http://coffeescript.org/) käyttämisen, tällöin tiedostojen päätteeksi tulee <code>.js.coffee</code>. Sovellusta suoritettaessa scprockets kääntää coffeescriptin automaattisesti javascriptiksi.

Tuotantokäytössä sovelluksella ei suorituskykysyistä yleensä kannata olla useampia javascript- tai tyylitiedostoja. Kun sovellusta aletaan suorittaa tuotantoympäristössä (eli production-moodissa), sprockets yhdistääkin kaikki sovelluksen javascript- ja tyylitiedostot yksittäisiksi, optimoiduiksi tiedostoiksi. Huomaamme tämän jos katsomme herokussa olevan sovelluksen html-lähdekoodia, esim: http://wad-ratebeer.herokuapp.com/ sisältää se nyt ainoastaan yhden js- ja yhden css-tiedoston joista varsinkin js-tiedoston luettavuus on ihmisen kannalta heikko.

Lisää asset pipelinestä ja mm. javascriptin liittämisestä railssovelluksiin mm. seuraavissa:
* http://railscasts.com/episodes/279-understanding-the-asset-pipeline
* http://railsapps.github.io/rails-javascript-include-external.html


> ## Tehtävät 7-9 (kolmen tehtävän arvoinen)
>
> ### Tehtävä on hieman työläs, joten tee ensin helpommat pois alta. Muut viikon tehtävät eivät riipu tästä tehtävästä.
>
> Toistaiseksi kuka tahansa voi sovelluksessamme liittyä olutkerhon jäseneksi. Muutetaan nyt sovellusta siten, että jäsenyys ei tule voimaan ennenkuin joku jo jäsenenä oleva vahvistaa jäsenyyden.
>
> Muutamia huomioita
> * jäsenyyden vahvistamattomuus kannattaa huomioida siten, että Membership-modeliin lisätään boolean-arvoinen kenttä _confirmed_
> * Kun kerho luodaan, tee sen luoneesta käyttäjästä automaattisesti kerhon jäsen
> * Näytä kerhon sivulla jäsenille lista vahvistamattomana olevista jäsenyyksistä (eli jäsenhakemuksista)
> * Jäsenyyden statuksen muutos voidaan hoitaa esim. oman [custom-reitin](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko6.md#reitti-panimon-statuksen-muuttamiselle) avulla.
>
> Tehtävä saattaa olla hieman haastava. [Active Record Associations -guiden](http://guides.rubyonrails.org/association_basics.html) luku **4.3.3 Scopes for has_many** tarjoaa erään hyvän työvälineen tehtävään. Tehtävän voi toki tehdä monella muullakin tavalla.
> Myös luku **4.3.2.3 :class_name** voi olla hyödyksi.

Tehtävän jälkeen sovelluksesi voi näyttää esim. seuraavalta. Olutseuran sivulla näytetään lista jäsenyyttä hakeneista, jos kirjautuneena on olutseurassa jo jäsenenä oleva käyttäjä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-6.png)

Käyttäjän omalla sivulta näytetään toistaiseksi käsittelemättömät hakemukset:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-5.png)

## Indeksi tietokantaan

Kun käyttäjä kirjautuu järjestelmäämme, suoritetaan sessiokontrollerissa operaatio, jossa käyttäjäolio haetaan tietokannasta käyttäjän nimen perusteella:

```ruby
class SessionsController < ApplicationController
  def create
    user = User.find_by username: params[:username]

     # ...
  end

end
```

Operaation suorittamista varten tietokanta joutuu käymään läpi koko <code>users</code>-taulun. Haut olion id:n suhteen ovat nopeampia, sillä jokainen taulu on indeksöity id:iden suhteen. Indeksi toimii hajautustaulun tavoin, eli tarjoaa "O(1)"-ajassa toimivan pääsyn haettuun tietokannan riviin.

Tietokantojen tauluihin voidaan lisätä tarvittaessa muitakin indeksejä. Nopeutetaan <code>users</code>-taulusta tapahtuvaa käyttäjätunnuksen perusteella tehtävää hakua lisäämällä taululle indeksi.

Luodaan indeksiä varten migraatio

    rails g migration AddUserIndexBasedOnUsername

Migraatio on seuraavanlainen:

```ruby
class AddUserIndexBasedOnUsername < ActiveRecord::Migration
  def change
    add_index :users, :username
  end
end
```

Suoritetaan migraatio komennolla <code>rake db:migrate</code> ja indeksi on valmis!

Indeksin huono puoli on se, että kun järjestelmään lisätään uusi käyttäjä tai olemassaoleva käyttäjä poistetaan, on indeksiä muokattava ja tähän luonnollisestsi kuluu aikaa. Indeksin lisäys on siis tradeoff sen suhteen, mitä operaatiota halutaan optimoida.

## Laiska lataaminen, n+1-ongelma ja tietokantakyselyjen optimointi

Kaikki oluet näyttävä kontrolleri on yksinkertainen. Oluet haetaan tietokannasta, järjestetään HTTP-kutsussa olleen parametrin määrittelemällä tavalla ja asetetaan templatea varten muuttujaan:

```ruby
  def index
    @beers = Beer.all

    order = params[:order] || 'name'

    @beers = case order
      when 'name' then @beers.sort_by{ |b| b.name }
      when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
      when 'style' then @beers.sort_by{ |b| b.style.name }
    end
  end
```

Template listaa oluet taulukkona:

```erb
<% @beers.each do |beer| %>
  <tr>
    <td><%= link_to beer.name, beer %></td>
    <td><%= link_to beer.style, beer.style %></td>
    <td><%= link_to beer.brewery.name, beer.brewery %></td>
  </tr>
<% end %>
</table>
```

Yksinkertaista ja tyylikästä... mutta ei kovin tehokasta.


Voisimme katsoa lokitiedostosta log/development.log mitä kaikkea oluiden sivulle mentäessä tapahtuu. Pääsemme samaan tietoon hieman mukavammassa muodossa käsiksi _miniprofiler_ gemin (ks. https://github.com/MiniProfiler/rack-mini-profiler ja http://samsaffron.com/archive/2012/07/12/miniprofiler-ruby-edition)

Miniprofilerin käyttöönotto on helppoa, riittää että Gemfileen lisätään rivi

    gem 'rack-mini-profiler'

Suorita <code>bundle install</code> ja käynnistä rails server uudelleen. Kun menet tämän jälkeen osoitteeseen http:localhost:300/beers huomaat, että sivun yläkulmaan ilmestyy aikalukema joka kuvaa HTTP-pyynnön suoritukseen käytettyä aikaa. Numeroa klikkaamalla avautuu tarkempi erittely ajankäytöstä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-7.png)

Raportti kertoo että <code>Executing action: index</code> eli kontrollerimetodin suoritus aiheuttaa yhden SQL-kyselyn <code>SELECT "beers".* FROM "beers"</code>. Sen sijaan
 <code>Rendering: beers/index</code> eli näkymätemplaten suoritus aiheuttaa peräti 9 SQL-kyselyä!

Kyselyjä klikkaamalla päästään tarkastelemaan syytä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-8.png)

Näkymätemplaten renderöinti siis suorittaa useaan kertaan seuraavat kyselyt:

```ruby
SELECT  "breweries".* FROM "breweries"  WHERE "breweries"."id" = ?  ORDER BY "breweries"."id" ASC LIMIT 1

SELECT  "styles".* FROM "styles"  WHERE "styles"."id" = ?  ORDER BY "styles"."id" ASC LIMIT 1
```
Käytännössä jokaista erillistä olutta kohti tehdään oma kysely sekä <code>styles</code>- että <code>breweries</code> tauluun.

Syynä tälle on se, että activerecordissa on oletusarvoisesti käytössä ns. _lazy loading_, eli kun haemme olion tietokannasta, olioon liittyvät kentät haetaan tietokannasta vasta jos niihin viitataan. Joskus tämä käyttäytyminen on toivottavaa, olioonhan voi liittyä suuri määrä olioita, joita ei välttämättä tarvita olion itsensä käsittelyn yhteydessä. Kaikkien oluiden sivulle mentäessä lazy loading ei kuitenkaan ole hyvä idea, sillä tiedämme varmuudella että jokaisen oluen yhteydessä näytetään myös oluen panimon sekä tyylin nimet ja nämä tiedot löytyvät ainoastaan panimoiden ja tyylien tietokantatauluista.

Voimme ohjata ActiveRecordin metodien parametrien avulla kyselyistä generoituvaa SQL:ää. Esim. seuraavasti voimme ohjeistaa, että oluiden lisäksi niihin liittyvät panimot tulee hakea tietokannasta:

```ruby
  def index
    @beers = Beer.includes(:brewery).all
    # ...
  end
```

Miniprofilerin avulla näemme. että kontrollerin suoritus aiheuttaa nyt kaksi kyselyä:

```ruby
SELECT "beers".* FROM "beers"
SELECT "breweries".* FROM "breweries"  WHERE "breweries"."id" IN (1, 2, 3, 6)
```

Näyttötemplaten  suoritus aiheuttaa enää 5 kyselyä, jotka kaikki ovat muotoa:

```ruby

SELECT  "styles".* FROM "styles"  WHERE "styles"."id" = ?  ORDER BY "styles"."id" ASC LIMIT 1
```

Näytön renderöinnin yhteydessä enää on haettava oluisiin liittyvät tyylit tietokannasta, kukin omalla SQL-kyselyllä.

Optimoidaan kontrolleria vielä siten, että myös kaikki tarvittavat tyylit luetaan kerralla kannasta:

```ruby
  def index
    @beers = Beer.includes(:brewery, :style).all

    # ...
  end
```

Kontrollerin suoritus aiheuttaa nyt kolme kyselyä ja näytön renderöinti ainoastaan yhden kyselyn. Miniprofiler paljastaa että kysely on

```ruby
SELECT  "users".* FROM "users"  WHERE "users"."id" = ? LIMIT 1
```

ja syynä sille on

```ruby
app/controllers/application_controller.rb:10:in 'current_user'
```
eli näytön muuttujan <code>current_user</code> avulla tekemä viittaus kirjautuneena olevaan käyttäjään. Tämä ei kuitenkaan ole hirveän vakavaa.

Saimme siis optimoitua SQL-kutsujen määrän 1+2n:stä (missä n tietokannassa olevien oluiden määrä) kolmeen (plus yhteen)!

Kokemaamme kutsutaan n+1-ongelmaksi (ks. http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations), eli hakiessamme kannasta yhdellä kyselyllä listallisen olioita, jokainen listan olioista aiheuttaakin salakavalasti uuden tietokantahaun ja näin yhden haun sijaan tapahtuukin noin n+1 hakua.

Muutetaan seuraavaa tehtävää varten kaikki käyttäjät listaava template  muotoon

```ruby
<h1>Users</h1>

<table class="table table-hover">
  <thead>
    <tr>
      <th>Username</th>
      <th> rated beers </th>
      <th> total ratings </th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <% @users.each do |user| %>
      <tr>
        <td><%= link_to user.username, user %></td>
        <td><%= user.beers.size %></td>
        <td><%= user.ratings.size %></td>
        <td>
          <% if admin_user and user.frozen? %>
            <span class="label label-info">account frozen</span>
          <% end %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Huomaa, että elementin <code>td</code> sisällä olevan if:in ehdon toimivuus riippuu siitä miten olet nimennyt asioita viikolla 5 tehdyn tehtävän koodissa. Voit tarvittaessa poistaa koko ehdon.

> ## Tehtävä 10
>
> Muutos aiheuttaa n+1-ongelman käyttäjien sivulle. Korjaa ongelma edellisen esimerkin tapaan eager loadaamalla tarvittavat oliot käyttäjien hakemisen yhteydessä. Varmista optimointisi onnistuminen miniprofilerilla.

**Huom:** jos taulukkoon liitettäisi myös suosikkioluen kertova sarake

     <td><%= user.favorite_beer %></td>

Muuttuisi tilanne hieman hankalammaksi SQL:n optimoinnin suhteen. Metodimme viimeisin versio oli seuraava:

```ruby
  def favorite_beer
    return nil if ratings.empty?
    ratings.order(score: :desc).limit(1).first.beer
  end
```

Nyt edes eager loadaaminen ei auta, sillä metodikutsu auheuttaa joka tapauksessa SQL-kyselyn. Jos sen sijaan toteuttaisimme metodin keskusmuistissa olueeseen liittyviä reittauksia (kuten teimme aluksi viikolla 4):

```ruby
  def favorite_beer
    return nil if ratings.empty?
    ratings.sort_by{ |r| r.score }.last.beer
  end
```

metodikutsu _ei_ aiheuttaisi tietokantaoperaatiota _jos_ reittaukset olisi eager loadattu siinä vaiheessa kun metodia kutsutaan.

Saattaakin olla, että metodista olisi tietyissä tilanteissa suorituskykyä optimoitaessa hyvä olla kaksi versiota, toinen joka suorittaa operaation tietokantatasolla ja toinen keskusmuistissa operaation tekevä.

## Cachays eli palvelinpuolen välimuistitoiminnallisuudet

Luodaan tietokantaamme hiukan lisää dataa. Kopioi seuraava tiedostoon db/seeds.db

```ruby
users = 200           # jos koneesi on hidas, riittää esim 100
breweries = 100       # jos koneesi on hidas, riittää esim 50
beers_in_brewery = 40
ratings_per_user = 30

(1..users).each do |i|
  User.create! username:"user_#{i}", password:"Passwd1", password_confirmation:"Passwd1"
end

(1..breweries).each do |i|
  Brewery.create! name:"Brewery_#{i}", year:1900, active:true
end

bulk = Style.create! name:"Bulk", description:"cheap, not much taste"

Brewery.all.each do |b|
  n = rand(beers_in_brewery)
  (1..n).each do |i|
    beer = Beer.create! name:"Beer #{b.id} -- #{i}", style:bulk
    b.beers << beer
  end
end

User.all.each do |u|
  n = rand(ratings_per_user)
  beers = Beer.all.shuffle
  (1..n).each do |i|
    r = Rating.new score:(1+rand(50))
    beers[i].ratings << r
    u.ratings << r
  end
end
```

Käytämme tiedostossa normaalien olioiden luovien metodien <code>create</code> sijaan huutomerkillistä versiota <code>create!</code>. Metodien erona on niiden käyttäytyminen tilanteessa, jossa olion luominen ei onnistu. Huutomerkitön metodi palauttaa tällöin arvon <code>nil</code>, huutomerkillinen taas aiheuttaa poikkeuksen. Seedauksessa poikkeuksen aiheuttaminen on parempi vaihtoehto, muuten luomisen epäonnistuminen jää herkästi huomaamatta.

**Kopioi sitten vanha tietokanta _db/development.sqlite_ talteen**, jotta voit palata vanhaan tilanteeseen suorituskyvyn virittelyn jälkeen. Vot ottaa vanhan tietokannan käyttöön muuttamalla sen nimeksi jälleen development.sqlite

**Huom:** tämä ei ole välttämättä paras mahdollinen tapa tehdä suorituskykytestausta oikeille Rails-sovelluksille, ks. lisää tietoa seuraavasta http://guides.rubyonrails.org/v3.2.13/performance_testing.html (guidesta ei ole Rails 4:lle päivitettyä versiota.)

Suorita seedaus komennolla

    rake db:seed

Skriptin suorittamisessa kuluu tovi.

**Huom:** jos skriptin suoritus päättyy virheeseen, kannattaa vian korjaamisen jälkeen palauttaa vanha tietokanta ennen skriptin uutta suorittamista. Eräs potentiaalinen ongelma skriptin suorituksessa on validoinnin rikkovat duplikaattinimet. Jos muutat komennon <code>create!</code> muotoon <code>create</code> ei skriptin suoritus keskeydy.

Nyt tietokannassamme on runsaasti dataa ja sivujen lataaminen alkaa olla hitaampaa.

Kokeile nyt miten sivujen suorituskykyyn vaikuttaa jos kommentoit pois äsken tekemäsi SQL-kyselyjen optimoinnit oluiden sivulta, eli muuta olutkontrolleri takaisin muotoon:

```ruby
  def index
    # @beers = Beer.includes(:brewery, :style).all
    @beers = Beer.all

    order = params[:order] || 'name'

    @beers = case order
      when 'name' then @beers.sort_by{ |b| b.name }
      when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
      when 'style' then @beers.sort_by{ |b| b.style.name }
    end
  end
```

Huomioi myös suoritettujen SQL-kyselyjen määrä optimoinnilla ja ilman. Tuloksenhan näet jälleen kätevästi miniprofilerin avulla.

Kokeilun jälkeen voit palauttaa koodin optimoituun muotoon.

Datamäärän ollessa suuri, ei pelkkä kyselyjen optimointi riitä, vaan on etsittävä muita keinoja.

Vaihtoehdoksi nousee tällöin **cachaus eli välimuistien käyttö**.

Web-sovelluksessa cachaystä voidaan suorittaa sekä selaimen, että palvelimen puolella (sekä selaimen ja palvelimen välissä olevissa proxyissä). Tarkastellaan nyt palvelimen puolella tapahtuvaa cachaystä. Toteutimme jo [toissa viikolla](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko5.md#suorituskyvyn-optimointi) "käsin" beermapping-apista haettujen tietojen cachaystä Rails.cachen avulla. Tutkitaan nyt muutamaa hieman automaattisempaa cachaysmekanismia.

Rails tarjoaa palvelinpuolen cachaystä kolmella tasolla:
* kokonaisten sivujen cachays (page cache)
* kontrollerin metodien tulosten cachays (action cache)
* sivufragmenttien cachays (fragment cache)

Ks. http://guides.rubyonrails.org/caching_with_rails.html

Rails 4:ssä kaksi ensimmäistä edellisistä on poistettu Railsin coresta mutta ne saadaan tarvittaessa käyttöön erillisten gemien avulla. Tutustummekin nyt ainoastaan viimeiseen eli **sivufragmenttien cachaamiseen**, joka onkin cachaysstrategioista monipuolisin.

Cachays ei ole oletusarvoisesti päällä kun sovellusta suoritetaan development-moodissa. Saat cachayksen päälle muuttamalla tiedostosta _config/environment/development.rb_ seuraavan rivin arvoksi <code>true</code>:

    config.action_controller.perform_caching = false

**Käynnistä nyt sovellus uudelleen.**

Päätetään cachata oluiden listan näyttäminen.

Fragmentticachays tapahtuu sisällyttämällä näkymätemplaten cachattavan osa seuraavanlaiseen lohkoon:

```erb
<% cache 'avain', skip_digest: true do %>
  cachättävä näkymätemplaten osa
<% end %>
```

Kuten arvata saattaa, <code>avain</code> on avain, jolla cachattava näkymäfragmentti talletetaan. Avaimena voi olla merkkijono tai olio .<code>skip_digest: true</code> liittyy [näyttötemplatejen versiointiin](http://blog.remarkablelabs.com/2012/12/russian-doll-caching-cache-digests-rails-4-countdown-to-2013) jonka haluamme nyt jättää huomioimatta. Tämä kuitenkin tarkoittaa, että välimuisti on syytä tyhjentää (komennolla <code>Rails.cache.clear</code>) _jos_ näkymätemplaten koodia muutetaan.

Fragmentticachayksen lisääminen oluiden listalle views/beers/index.html on helppoa, cachataan sivulta sen dynaaminen osa eli oluiden taulukko:

```erb
<h1>Beers</h1>

<% cache 'beerlist', skip_digest: true do %>

  <table class="table table-hover">
    <thead>
      <tr>
        <th> <%= link_to 'Name', beers_path(order:"name") %> </th>
        <th> <%= link_to 'Style', beers_path(order:"style") %> </th>
        <th> <%= link_to 'Brewery', beers_path(order:"brewery") %> </th>
      </tr>
    </thead>

    <tbody>
      <% @beers.each do |beer| %>
        <tr>
          <td><%= link_to beer.name, beer %></td>
          <td><%= link_to beer.style, beer.style %></td>
          <td><%= link_to beer.brewery.name, beer.brewery %></td>
        </tr>
      <% end %>
    </tbody>
  </table>

<% end %>

<br>

<%= link_to('New Beer', new_beer_path, class:'btn btn-primary') if current_user %>
```

Kun nyt menemme sivulle, ei sivufragmenttia ole vielä talletettu välimuistin ja sivun lataaminen kestää yhtä kauan kuin ennen cachayksen lisäämistä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-9.png)

Sivun renderöimiseen kulunut aika <code>Rendering: beers/index</code> oli siis 814 millisekuntia.

Sivulla käytyämme fragmentti tallettuu välimuistiin ja seuraava sivun avaaminen on huomattavasti nopeampi:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-10.png)

Eli näkymätemplate renderöityy kolmessa millisekunnissa!

Huom: uuden oluen luomislinkkiä ei kannata laittaa cachatyn fragmentin sisälle, sillä linkki tulee näyttää ainoastaan kirjautuneille käyttäjille. Sivun cachatty osa näytetään nyt kaikille samanlaisena.

Jos luomme nyt uuden oluen huomaamme, että uuden oluen tietoja ei tule sivulle. Syynä tälle on tietenkin se, että sivufragmentti löytyy edelleen välimuistista. Vanhentunut näkymäfragmentti tulisi siis ekspiroida. Tässä tapauksessa helpoin strategia on kontrollerista käsin tapahtuva manuaalinen ekspirointi.

Ekspirointi tapahtuu komennolla <code>expire_fragment(avain)</code> jota tulee siis kutsua kontrollerista niissä kohdissa joissa oluiden listan sisältö mahdollisesti muuttuu. Tälläisiä kohtia ovat olutkontrollerin metodit <code>create</code>, <code>update</code> ja <code>destroy</code>. Muutos on helppo:

```erb
  def create
    expire_fragment('beerlist')
  end

  def update
    expire_fragment('beerlist')
    # ...
  end

  def destroy
    expire_fragment('beerlist')
    # ...
  end
```

Muutosten jälkeen sivu toimii odotetulla tavalla!

Kaikkien oluiden sivua olisi mahdollista nopeuttaa vielä jonkin verran. Nyt nimittäin kontrolleri suorittaa tietokantaoperaation

```erb
    @beers = Beer.includes(:brewery, :style).all
```

myös silloin kun sivufragmentti löytyy välimuistista. Voisimmekin testata fragmentin olemassaoloa metodilla <code>fragment_exist?</code> ja suorittaa tietokantaoperaation ainoastaan jos fragmentti ei ole olemassa (muistutus: unless tarkoittaa samaa kuin if not):

```erb
  def index
    unless fragment_exist?( 'beerlist' )
      @beers = Beer.includes(:brewery, :style).all

      order = params[:order] || 'name'

      @beers = case order
        when 'name' then @beers.sort_by{ |b| b.name }
        when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
        when 'style' then @beers.sort_by{ |b| b.style.name }
      end
    end
  end
```

Kontrolleri näyttää hieman ikävältä, mutta sivu nopeutuu entisestään:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w6-11.png)

Voisimme siistiä ratkaisua aavistuksen palauttamalla metodin <code>index</code> ennalleen ja tarkastamalla fragmentin olemassaolon esifiltterissä:

```ruby
  before_action :skip_if_cached, only:[:index]

  def skip_if_cached
    return render :index if fragment_exist?( 'beerlist' )
  end
```

Jos esifiltteri <code>skip_if_cached</code> huomaa, että fragmentti on olemassa, se renderöi näkymätemplaten suoraan. Tässä tapauksessa varsinaista kontrollerimetodia ei suoriteta ollenkaan.

Huomaamme kuitenkin että sivulla on pieni ongelma. Oluet sai järjestettyä sarakkeita klikkaamalla vaihtoehtoisiin järjestyksiin. Cachays on kuitenkin rikkonut toiminnon!

Yksi tapa korjata toiminnallisuus on liittää fragmentin avaimeen järjestys:

```erb
<% cache "beerlist-#{@order}", skip_digest: true do %>
   taulukon html
<% end %>
```

Järjestys talletetaan siis muuttujaan <code>@order</code> kontrollerissa. Seuraavassa kontrollerin vaatimat muutokset:

```ruby
  def skip_if_cached
    @order = params[:order] || 'name'
    return render :index if fragment_exist?( "beerlist-#{@order}"  )
  end

  def index
    @beers = Beer.includes(:brewery, :style).all

    @beers = case @order
      when 'name' then @beers.sort_by{ |b| b.name }
      when 'brewery' then @beers.sort_by{ |b| b.brewery.name }
      when 'style' then @beers.sort_by{ |b| b.style.name }
    end
  end
```

eli koska kontrollerimetodia ei välttämättä suoriteta, tallettaa esifiltteri järjestyksen.

Ekspiroinnin yhteydessä on ekspiroitava kaikki kolme järjestystä:

```ruby
   ["beerlist-name", "beerlist-brewery", "beerlist-style"].each{ |f| expire_fragment(f) }
```

**Huom:** voit kutsua fragmentticachen operaatioita konsolista:

```ruby
> ActionController::Base.new.fragment_exist?( 'beerlist-name' )
Exist fragment? views/beerlist-name (0.4ms)
=> true
> ActionController::Base.new.expire_fragment( 'beerlist-name' )
Expire fragment views/beerlist-name (0.6ms)
=> true
> ActionController::Base.new.fragment_exist?( 'beerlist-name' )
Exist fragment? views/beerlist-name (0.1ms)
=> nil
```

**Huom2:** konsoliakin käytevämpi debuggauskeino on liittää kehityksen aikana sivun cachaamattomaan osaan tieto siitä mitä fragmentteja cachesta löytyy ja mikä on nykyistä sivua vastaava fragmentti. Lisätään seuraava oluiden sivun yläosaan:

```html
<div style="border-style: solid;">
  beerlist-name: <%= ActionController::Base.new.fragment_exist?( 'beerlist-name' ) %>
  <br>
  beerlist-style: <%= ActionController::Base.new.fragment_exist?( 'beerlist-style' ) %>
  <br>
  beerlist-brewery: <%= ActionController::Base.new.fragment_exist?( 'beerlist-brewery' ) %>
  <br>
  current: <%= "beerlist-#{@order}" %>
</div>
```

Nyt sivun yläreunaan tulee laatikko, joka kertoo välimuistifragmenttien tilan, eli onko fragmentti olemassa vai ei:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2017/raw/master/images/ratebeer-w7-6.png)

**Huom3:** koska debuglaatikko on sivun yläreunassa eli ennen  fragmentin mahdollisesti generoivaa koodia, **tulee sivu aina reloadata**, jotta debuglaatikko näyttäisi fragmenttien ajantasaisen tilanteen.

> ## Tehtävä 11
>
> Toteuta panimot listaavalle sivulle fragmentticachays. Varmista, että sivun sisältöön vaikuttava muutos ekspiroi cachen. Voit jättää huomiotta tehtävässä 2 tehdyn lisäyksen, jonka avulla järjestys saadaan muutettua päinvastaiseksi klikkaamalla sarakkeen nimeä uudelleen.

## yksittäisen oluen sivun cashays

Jos haluaisimme cachata yksittäisen oluen sivun, kannattaa fragmentin avaimeksi laittaa itse cachattava olio:

```ruby
<% cache @beer do %>

  <h2>
    <%= @beer.name %>
  </h2>

  <p>
    <%= link_to @beer.style, @beer.style %>
    brewed by
    <%= link_to @beer.brewery.name, @beer.brewery %>
  </p>

  <% if @beer.ratings.empty? %>
      <p>beer has not yet been rated! </p>
  <% else %>
      <p>Has <%= pluralize(@beer.ratings.count,'rating') %>, average <%= round(@beer.average_rating) %> </p>
  <% end %>

<% end %>

<!- cachaamaton osa >

<% if current_user %>

  <h4>give a rating:</h4>

  <%= form_for(@rating) do |f| %>
      <%= f.hidden_field :beer_id %>
      score: <%= f.number_field :score %>
      <%= f.submit class:"btn btn-primary"  %>
  <% end %>

  <p></p>

<% end %>

<%= edit_and_destroy_buttons(@beer) %>
```

Nyt fragmentin avaimeksi tulee merkkijono, jonka Rails generoi kutsumalla olion metodia <code>cache_key</code>. Metodi generoi avaimen joka yksilöi olion _ja_ sisältää aikaleiman, joka edustaa hetkeä, jolloin olio on viimeksi muuttunut. Jos olion kenttiin tulee muutos, muuttuu fragmentin avaimen arvo eli vanha fragmentti ekspiroituu automaattisesti. Seuraavassa esimerkki automaattisesti generoituvasta cache-avaimesta:

```ruby
> b = Beer.first
> b.cache_key
=> "beers/1-20170106210715764510000"
> b.update_attribute(:name, 'ISO 4')
> b.cache_key
=> "beers/1-20170211200158000903000"
```

Ratkaisu on vielä sikäli puutteellinen, että jos olueeseen tehdään uusi reittaus, olio ei itsessään muutu ja fragmentti ei ekspiroidu. Ongelma on kuitenkin helppo korjata. Lisätään reittaukseen tieto, että reittauksen syntyessä, muuttuessa tai tuhoutuessa, on samalla 'kosketettava' reittaukseen liittyvää olutta:

```ruby
class Rating < ActiveRecord::Base
  belongs_to :beer, touch: true

  # ...
end
```

Käytännössä <code>belongs_to</code>-yhteyteen liitetty <code>touch: true</code> saa aikaan sen, että yhteyden toisessa päässä olevan olion kenttä <code>updated_at</code> päivittyy.

> ## Tehtävä 12
>
> Toteuta yksittäisen panimon sivulle fragmentticachays. Huomaa, että edellisen esimerkin tapaan panimon sivufragmentin on ekspiroiduttava automaattisesti jos panimon oluisiin tulee muutoksia.

Välimuistin eksplisiittinen ekspiroiminen, kuten kaikkien oluiden sivun suhteen joudumme tekemään, on hieman ikävää sillä on aina pieni riski, että koodissa ei muisteta ekspiroida fragmenttia kaikissa tarpeellisissa kohdissa.

Käyttäessämme suoraan olioa (kuten yksittäisen oluen sivulla tehtiin) fragmentin avaimena, ekspiroitui cache automaattisesti olion päivittyessä. Myös kaikkien oluiden sivulle olisi mahdollista tehdä automaattisesti ekspiroituva cache generoimalla fragmentin avain tarkoitukseen sopivan metodin avulla, katso
[Caching with Rails: An overview](http://guides.rubyonrails.org/caching_with_rails.html#fragment-caching) -guiden
kohta "If you want to avoid expiring the fragment manually..."

## Selainpuolen cachays

Cachaysta harjoitetaan monilla tasoilla, myös selainpuolella. Suosittelen aiheesta kiinnostuneille Codeschoolin videota [etag](https://en.wikipedia.org/wiki/HTTP_ETag):ien käytöstä Rails-sovelluksissa, ks. http://rails4.codeschool.com/videos

Codeschoolin muutkin Rails 4 -videot ovat erittäin suositeltavia!

## Eventual consistency

Sovelluksen käyttäjän kannalta ei ole aina välttämätöntä, että sovelluksen näyttämä tilanne on täysin ajantasalla. Esim. ei ole kriittistä, jos reittausstatistiikkaa näyttävä ratings-sivu näyttää muutaman minuutin vanhan tilanteen, pääasia on, että kaikki järjestelmään tullut data tulee ennen pitkää näkyville käyttäjille. Tälläisestä hieman löyhemmästä ajantasaisuusvaatimuksesta, jolla saatetaan pystyä tehostamaan sovelluksen suorituskykyä huomattavasti käytetään englanninkielistä nimitystä [__eventual consistency__](http://en.wikipedia.org/wiki/Eventual_consistency).

Eventual consistency -mallin mukainen ajantasaisuus on helppo määritellä Railsissa laittamalla esim. fragmentticachelle expiroitumisaika:

```erb
<% cache 'fragment_name', expires_in:10.minutes do %>
  ...
<% end %>
```

Tämä yksinkertaistaa sovellusta myös siinä mielessä, että cachen ekspiroiminen on helppoa, kun ei ole pakko huomioida kaikkia yksittäisiä kohtia koodissa, jotka voisivat aiheuttaa sivun epäajantasaisuuden.

Oletetaan, että järjestelmällämme olisi todella paljon käyttäjiä ja reittauksia tapahtuisi useita kertoja minuutissa. Jos haluaisimme näyttää ratings-sivulla täysin ajantasaista tietoa, ei sivun suorituskyky olisi hyvä, sillä jokainen oluen reittaus muuttaa sivun tilaa, ja sivu tulisi ekspiroida erittäin usein. Tämä taas tekisi cachayksestä lähes hyödyttömän.

SQL:n optimointi ja cachayskään eivät vielä tee ratings-sivusta kovin nopeita, sillä kontrollerin käyttämät operaatiot esim. <code>User.top(3)</code> vaativat käytännössä melkein koko tietokannan datan läpikäyntiä. Jos haluaisimme optimoida sivua vielä enemmän, tulisi meidän käyttää järeämpiä keinoja. Esim <code>User.top</code>-komennon suoritus nopeutuisi huomattavasti, jos käyttäjän reittausten määrä talletettaisiin suoraan käyttäjä-olioon, eli sen laskeminen ei edellyttäisi käyttäjään liittyvien Rating-olioiden lukumäärän laskemista. Tämä taas edellyttäisi, että aina käyttäjän uuden reittauksen yhteydessä päivitettäisiin myös käyttäjä-olioa. Eli itse reittausoperaation suoritus hidastuisi hieman.

Toinen ja ehkä parempi tapa reittaussivun nopeuttamiselle olisi cachata Rails.cacheen kontrollerin tarvitsemat tiedot. Kontrolleri on siis seuraava

```ruby
  def index
    @top_beers = Beer.top(3)
    @top_breweries = Brewery.top(3)
    @top_styles = Style.top(3)
    @most_active_users = User.most_active(3)
    @recent_ratings = Rating.recent
    @ratings = Rating.all
  end
```

Voisimme toimia nyt samoin kuin viikolla 5 kun talletimme olutravintoloiden tietoja Railsin cacheen, eli kontrolleri muuttuisi suunnilleen seuraavaan muotoon:

```ruby
  def index
    Rails.cache.write("beer top 3", Beer.top(3)) if cache_does_not_contain_data_or_it_is_too_old
    @top_beers = Rails.cache.read "beer top 3"

    # ...
  end
```

Kurssin tehtäväkirjanpitosovelluksen etusivu http://wadrorstats2017.herokuapp.com/ toimii tällä hetkellä melko nopeasti vaikka sivulla näytetäänkin kaikista järjestelmään tehdyistä palautuksista koostetut tilastot. Palautuksia on tällä hetkellä yli 600. Etusivun suorituskyky ei muuttuisi oikeastaan ollenkaan vaikka palautuksia olisi tuhatkertainen määrä.

Sovellus tallettaa jokaisen palautuksen <code>Submission</code>-olioon. Jos etusivun generoinnin yhteydessä palautustilastot laskettaisiin käymällä läpi kaikki <code>Submission</code>-oliot, olisi sivun renderöinti huomattavasti hitaampaa, ja se hidastuisi sitä mukaa kun järjestelmään tulisi enemmän palautuksia.

Sovellus toimii kuitenkin siten, että jokaisen viikon palautustilastot on esilaskettu <code>WeekStatistic</code>-olioihin. Sovellus päivittää viikkostatistiikkaa jokaisen uuden tai päivitetyn palautuksen yhteydessä. Tämä hidastaa palautuksen tekoa mariginaalisesti (operaatiossa pullonkaulana on sähköpostin lähetys, lähetykseen kuluu 95% operaation vievästä ajasta), mutta nopeuttaa etusivun lataamista todella paljon. Eli viikolla 7 etusivun tiedot saadaan generoitua seitsemän olion perusteella ja oliot saadaan tietokannasta yhdellä SQL-kyselyllä ja käytännössä etusivun generointi on suorituskyvyltään täysin riippumaton tehtyjen palautusten määrästä.

Sovelluksen koodi löytyy osoitteesta
https://github.com/mluukkai/wadrorstats

Sovellusten suorituskyvyn optimointi ei ole välttämättä helppoa, se edellyttää monentasoisia ratkaisuja ja pitää useimmiten tehdä tilannekohtaisesti räätälöiden. Koodi muuttuu yleensä optimoinnin takia rumemmaksi.

## Asynkronisuus, viestijonot ja taustatyöt

Yhtenä negatiivisena puolena cachen ajoittain tapahtuvassa ekspiroimisessa, esim. jos noudattaisimme strategiaa ratings-sivun suhteen, aiheutuu jollekin käyttäjälle aika ajoin paljon aikaavievä operaatio siinä vaiheessa kun data on generoitava uudelleen välimuistiin.

Parempaan ratkaisuun päästäisiinkin jos käyttäjälle tarjottaisiin aina niin ajantasainen data kuin mahdollista, eli kontrolleri olisi muotoa:

```ruby
  def index
    @top_beers = Rails.cache.read "beer top 3"

    # ...
  end
```

Välimuistin päivitys voitaisiin sitten suorittaa omassa taustalla olevassa, aika ajoin heräävässä säikeessä/prosessissa:

```ruby
  # pseudokoodia, ei toimi oikeasti...
  def background_worker
    while true do
       sleep 10.minutes
       Rails.cache.write("beer top 3", Beer.top(3))
       Rails.cache.write("brewery top 3", Brewery.top(3))
       # ...
    end
  end
```

Ylläesitellyn kaltainen taustaprosessointitapa on siinä mielessä yksinkertainen, että sovelluksen ja taustaprosessointia suorittavan säikeen/prosessin ei tarvitse synkronoida toimintojaan. Toisinaan taas taustaprosessoinnin tarpeen laukaisee jokin sovellukselle tuleva pyyntö. Tällöin sovelluksen ja taustaprosessoijien välisen synkronoinnin voi hoitaa esim. viestijonojen avulla.

Viestijonoilla ja erillisillä prosesseilla tai säikeillä hoidetun taustaprosessoinnin toteuttamiseen Railsissa on paljon erilaisia vaihtoehtoja, tämän hetken paras ratkaisu näistä on [Sidekiq](http://railscasts.com/episodes/366-sidekiq). 

Jos sovellus tarvitsee ainoastaan jonkin yksinkertaisen, tasaisin aikavälein suoritettavan taustaoperaation, saattaa [Heroku scheduler](https://devcenter.heroku.com/articles/scheduler) olla yksinkertaisin vaihtoehto. Tällöin taustaoperaatio määritellään [Rake-taskina](http://railscasts.com/episodes/66-custom-rake-tasks), jonka Heroku suorittaa joko kerran vuorokaudessa, tunnissa tai kymmenessä minuutissa.

Ennen Rails 4:sta Rails-sovellukset toimivat oletusarvoisesti yksisäikeisinä. Tästä taas oli seurauksena se, että sovellus käsitteli HTTP-pyynnöt peräkkäin (ellei palvelinohjelmiston tasolla oltu määritelty että sovelluksesta on käynnissä useampia instansseja, ks. esim. https://devcenter.heroku.com/articles/rails-unicorn). Rails 4:stä asti jokaisen pyynnön käsittely omassa säikeessään on oletusarvoisesti sallittu. On kuitenkin huomattava, että Railsin oletusarvoinen palvelinohjelmisto WEBrick _ei_ tue säikeistettyä pyyntöjen käsittelyä. Jos säikeistykselle on tarvetta, tulee palvelinohjelmistona käyttää [Pumaa](http://puma.io/). Puman käyttöönotto on [helppo tehdä](https://discussion.heroku.com/t/using-puma-on-heroku/150).

Lisää säikeistettyjen Rails-sovellusten tekemisestä [Rails-castista] (https://www.cs.helsinki.fi/i/mluukkai/365-thread-safety.mp4). Huomaa, että säikeistyksen sallimisen jälkeen on huolehdittava siitä että koodi on säieturvallista!

## Sucker Punch

Kuten edellä todettiin, paras vaihtoehto asynkronisten operaatioiden suorittamiseen Railsilla on [Sidekiq](http://railscasts.com/episodes/366-sidekiq). Sidekiq kuitenkin vaatii oman prosessinsta, eli esim. Herokussa sidekiqia ei ole helppoa suorittaa varaamatta sille omaa prosessia eli [dynoa](https://devcenter.heroku.com/articles/dynos), ja se taas maksaa vähintään 7 dollaria kuussa.

Ilmaisten Heroku-palveluiden yhteydessä on madollista [Sucker Punch](https://github.com/brandonhilkert/sucker_punch)- kirjastoa:

> Sucker Punch is a single-process Ruby asynchronous processing library. This reduces costs of hosting on a service like Heroku along with the memory footprint of having to maintain additional jobs if hosting on a dedicated server. All queues can run within a single application (eg. Rails, Sinatra, etc.) process.

Eli Sucker Punch suorittaa asynkroniset työt samassa prosessissa, missä itse Rails-sovellustakin suoritetaan. 

Sucker Punchin käyttö on melko helppoa. 

Lisää gemfileen <code>gem 'sucker_punch', '~> 2.0'</code> ja suorita bundle install.

Luodaan Sucker Punch -operaatioita varten hakemisto _app/jobs_ (Rails 5 luo hakemiston oletusarvoisesti). Listätään tiedostoon _application.rb_ seuraava rivi (Rails 5:llä tämä ei ole tarpeen):

```ruby
    config.autoload_paths += Dir["#{Rails.root}/app/jobs"]
```

eli määritellään Rails lataamaan automaattisesti luomaamme hakemistoon määritelty koodi.

Luodaan nyt Sucker Punch -operatio, eli tiedosto _test_job.rb_ jolla on seuraava sisältö:

```ruby
class TestJob
  include SuckerPunch::Job

  def perform
    puts "running job..."
  end
end
```

Voimme suorittaa operaation antamalla rails-konsolista (tai mistä tahansa kohtaa sovelluksen koodia) komennon

```ruby
TestJob.perform_async
```

Operaatio tulostaa konsoliin _running job..._. Ei kovin vakuuttavaa.

Huomionarvoista tässä on kuitenkin se, että operaatio suoritetaan asynkronisesti taustalla, eli kontrolli palaa konsoliin jo ennen kuin operaatio on suoritettu.

Muutetaan operaatiota seuraavasti:

```ruby
class TestJob
  include SuckerPunch::Job

  def perform
    sleep 1
    puts "starting job..."
    sleep 10
    puts "job ready!"
  end
end
```

eli nyt operaation suoritus kestää 11 sekuntia. Kun suoritat operaation komennolla <code>TestJob.perform_async</code> huomaat, että pääset takaisin konsoliin välittömästi komennon suorituksen jälkeen (joudut todennäköisesti painamaan enteriä että saat konsolin komentokehotteen näkyviin) ja operaation suoritus tapahtuu taustalla, samalla kun voit suorittaa konsolista halutessasi jotain muuta koodia.

Voit suorittaa operaation myös _synkronisesti_ antamalla komennon <code>TestJob.new.perform</code>. Tällöin joudut odottamaan komennon suorituksen loppuun asti ennen kuin konsoli aktivoituu uudelleen.

Voit myös suorittaa operaatioita ajastetusti, esim. jos annat komennon <code>TestJob.perform_in(10.seconds)</code> suoritetaan operaatio asynkronisesti 10 sekunnin kuluttua.

Asynkroninen operaatio voi käynnistää itse itsensä, eli jos muutat koodin muotoon 

```ruby
class TestJob
  include SuckerPunch::Job

  def perform
    sleep 1
    puts "starting job..."
    sleep 10
    puts "job ready!"
    TestJob.perform_in(30.seconds)
  end
end
```

ja annat komennon <code>TestJob.perform_async</code> operaatio suoritetaan toistuvasti 30 sekunin välein niin kauan kunnes konsoli suljetaan.

> ## Tehtävä 13
>
> Nopeuta ratings-sivun toimintaa haluamasi tekniikan. Voit olettaa, että käyttäjät ovat tyytyväisiä eventual consistency -mallin mukaiseen tiedon ajantasaisuuteen.  
>
>Kirjoita ratings-kontrollerin <code>index</code>-metodiin pieni selitys nopeutusstrategiastasi jos se ei ole koodin perusteella muuten ilmeistä.
>
> Jos päädyt käyttämään asynkronisia workereita, ei koodia ole välttämättä ihan helppoa saada toimimaan täysin oikein.

## Sovelluksen koostaminen palveluista

Sovelluksen suorituskyvyn skaalaaminen onnistuu vain tiettyyn pisteeseen asti, jos sovellus on monoliittinen, kokonaan yhden tietokannan varassa, yhdellä palvelimella suoritettava kokonaisuus. Sovellusta voidaan toki optimoida ja sitä voidaan skaalata __horisontaalisesti__ eli palvelimen fyysisiä resursseja kasvattamalla.

Parempaan skaalautuvuuteen päästään kuitenkin __vertikaalisella__ skaalautuvuudella, eli sen sijaan että palvelimen fyysisiä resursseja yritettäisiin kasvattaa, otetaankin sovelluksen käyttöön useita palvelimia, jotka suorittavat sovelluksen toimintoja rinnakkain. Vertikaalinen skaalaaminen ei välttämättä onnistu triviaalisti, sovelluksen arkkitehtuuria on mukautettava. Jos sovellusta palvelee edelleen ainoastaan yksi tietokanta, voi siitä tulla pullonkaula vertikaalisesta skaalaamisesta huolimatta, erityisesti jos kyseessä on relaatiotietokanta, joiden hajauttaminen ja näin ollen vertikaalinen skaalaaminen ei ole helppoa.

Sovelluksen skaalaaminen (ja joissain tapauksissa myös sen ylläpitäminen ja laajentaminen) on helpompaa, jos sovellus on koostettu useammista erillisistä itsenäisenä toimivista keskenään esim. HTTP-protokollan välityksellä kommunikoivista __palveluista__. Sovelluksemme itseasiassa hyödyntää jo toista palvelua eli BeermappingAPI:a. Vastaavasti sovelluksen toiminnallisuutta voitaisiin laajentaa integroimalla siihen uusia palveluja.

Jos haluaisimme esim. että sovelluksemme tekisi käyttäjälle suosikkioluttyyleihin ja sijaintiin (joka saadaan selvitettyä esim. käyttäjän tekemien HTTP-kutsujen IP-osotteen perusteella, ks http://www.iplocation.net/) perustuvia ruokareseptisuosituksia, kannattaisi suosittelijasta tehdä kokonaan oma palvelunsa. Sovelluksemme keskustelisi sitten palvelun kanssa HTTP-protokollaa käyttäen.

Jos haluaisimme vastaavasti, että sovelluksemme näyttäisi käyttäjälle olutsuosituksia käyttäjän oman suosikkityylin perusteella, olisi tämän toiminnallisuuden eriyttäminen omaksi, erillisellä palvelimella toimivaksi palveluksi hieman haastavampaa, sillä suositukset todennäköisesti riippuisivat muiden ihmisten tekemistä reittauksista ja tähän tietoon käsille pääsy taas edellyttäisi olutsuosittelijalta pääsyä sovelluksemme tietokantaan. Eli jos oluiden suosittelija haluttaisiin toteuttaa omana erillisenä palvelunaan, olisi sovelluksemme rakennetta kenties mietittävä kokonaan uudelleen, jotta tieto reittauksista saataisiin jaettua ratebeer-sovelluksen ja olutsuosituspalvelun kesken.

Viime aikoina onkin noussut suosioon tyyli, jossa sovellus koostetaan [mikropalveluista (engl. micro service)](https://martinfowler.com/articles/microservices.html) eli melko pienistä, yhden erillisen tehtävän itsenäisesti hoitavista palveluista. 

## Single sign on

Monilla sivustoilla on viime aikoina yleistynyt käytäntö, jossa mahdollistetaan sivulle kirjautuminen esim. Google-,  Facebook- tai GitHub-tunnuksilla. Sivustot siis ovat ulkoistaneet käyttäjänhallinnan ja autentikoinnin erillisille palveluille.

Autentikointi tapahtuu OAuth2-standardia (ks. https://tools.ietf.org/html/draft-ietf-oauth-v2-31) hyödyntäen, OAuth-autentikoinnin perusteista enemmän esim. osoitteessa http://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified

OAuth-pohjainen autentikaatio onnistuu Railsilla helposti Omniauth-gemien avulla, ks. http://www.omniauth.org/ Jokaista palveluntarjoajaa kohti on nykyään olemassa oma geminsä, esim. [omniauth-github](https://github.com/intridea/omniauth-github)

> ## Tehtävä 14
>
> Lisää sovellukseen mahdollisuus käyttää sitä GitHub-tunnuksilla. Etene seuraavasti:
> * Kirjaudu GitHubiin ja mene [setting-sivulle](https://github.com/settings/profile). Valitse vasemmalta _oauth applications_ ja klikkaa _Register new Application_, määrittele _homepage urliksi_ http://localhost:3000 ja _authorization callback urliksi_ http://localhost:3000/auth/github/callback
> * asenna [omniauth-github](https://github.com/intridea/omniauth-github) gem
> * Luo hakemistoon _config/initializers_ tiedosto _omniauth.rb_, jolla on seuraava sisältö:
> ```ruby
> Rails.application.config.middleware.use OmniAuth::Builder do
>  provider :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET']
> end
> ```
> * aseta GitHubiin luomasi sovelluksen sivulla olevat _client id_ ja _client secret_ edellä määriteltyjen [ympäristömuuttujien](https://github.com/mluukkai/WebPalvelinohjelmointi2017/blob/master/web/viikko5.md#sovelluskohtaisen-datan-tallentaminen) arvoksi
> * lisää tiedostoon _routes.rb_ reitti
> ```ruby
>   get 'auth/:provider/callback', to: 'sessions#create_oauth'
> ```
> * luo reitin määrittelemä kontrollerimetodi _sessiokontrolleriin_
> * tee sovellukseen nappi, jota klikkaamalla käyttäjä voi kirjautua sovellukseen GitHub-tunnuksilla. Napin pathi on _auth/github_
> * kun kirjaudut sovellukseesi GitHub-tunnuksilla, uudelleenohjautuu selain osoitteeseen _auth/github/callback_ eli routes.rb:n määrittelyn ansioista suoritus siirtyy sessiokontrollerin metodille _create_oauth_, pääset siellä käsiksi tarvittaviin tietoihin muuttujan <code>env["omniauth.auth"]</code> avulla:
> ```ruby
> (byebug) env["omniauth.auth"].info
#<OmniAuth::AuthHash::InfoHash email="mluukkai@iki.fi" image="https://avatars.githubusercontent.com/u/523235?v=3" name="Matti Luukkainen" nickname="mluukkai" urls=#<OmniAuth::AuthHash Blog=nil GitHub="https://github.com/mluukkai">>
(byebug)
> ```
> * tee sovellukset tarvittavat muutokset
> * kun sovellus toimii paikallisesti, vaihda GitHub-sovelluksen _homepage url_ ja _authorization callback url_ vastaamaan Herokussa olevan sovelluksesi urleja
>
> Muutokset eivät ole täysin suoraviivaisia:
> * sessiokontrollerin uuteen metodiin tulee kirjoittaa koodi, joka tarkastaa käyttäjän identiteetin ja luo tarvittaessa GitHub-käyttäjää vastaavan <code>User</code>-olion
> * joudut muokkaamaan <code>User</code>-modelia siten, että sen avulla hoidetaan sekä järjestelmän omaa salasanaa hyödyntävät käyttäjät, että GitHubin kautta kirjautuvat
> * tällä hetkellä <code>User</code>-olioiden validoinnissa vaaditaan, että olioilla on vähintään 4 merkin mittainen salasana. Joudut tekemään validoinnin ehdolliseksi, siten ettei sitä vaadita GitHubin tunnuksilla kirjautuvalta käyttäjältä (katso apua googlella) tai toinen vaihtoehto on generoida myös GitHubin kautta kirjautuville esim. satunnainen salasana

## NoSQL-tietokannat

Relaatiotietokannat ovat dominoineet tiedon tallennusta jo vuosikymmenten ajan. Viime aikoina on kuitenkin alkanut jälleen tapahtumaan tietokantarintamalla, ja kattotermin [NoSQL](https://en.wikipedia.org/wiki/NoSQL) alla kulkevat "ei relaatiotietokannat" ovat alkaneet nostaa suosiotaan.

Yhtenä motivaationa NoSQL-tietokannoilla on ollut se, että relaatiotietokantoja on vaikea skaalata massivisten internetsovellusten vaatimaan suorituskykyyn. Toisaalta myös tiettyjen NoSQL-tietokantojen skeemattomuus tarjoaa sovellukselle joustavuutta verrattuna SQL-tietokantojen tarkastimääriteltyihin tietokantaskeemoihin.

NoSQL-tietokantoja on useita, keskenään aivan erilaisilla toimintaperiaatteilla toimivia, mm.
* avain/arvotietokannat (key-value databases)
* dokumenttitietokannat (document databases)
* saraketietokannat (columnar databases)
* verkkotietokannat (graph databases)

Jo meille tutuksi tullut <code>Rails.cache</code> on oikeastaan yksinkertainen avain-arvotietokanta, joka mahdollistaa mielivaltaisten olioiden tallettamisen avaimeen perustuen. Tietokannasta haku on rajoittunut hakuun avaimien perusteella ja tietokanta ei tue kannassa olevien olioiden välisiä liitoksia ollenkaan.

Uusien tietokantatyyppien noususta huolimatta relaatiotietokannat tulevat kuitenkin säilymään ja on todennäköistä että isommissa sovelluksissa on käytössä rinnakkain erilaisia tietokantoja, ja kuhunkin talletustarkoitukseen pyritään valitsemaan tilanteeseen parhaiten sopiva tietokantatyyppi, ks.
http://www.martinfowler.com/bliki/PolyglotPersistence.html

> ## Tehtävä 15
>
> Kurssi on tehtävien osalta ohi ja on aika antaa kurssipalaute osoitteessa https://ilmo.cs.helsinki.fi/kurssit/servlet/Valinta

## Tehtävien palautus

Commitoi kaikki tekemäsi muutokset ja pushaa koodi GitHubiin. Deployaa myös uusin versio Herokuun.

Tehtävät kirjataan palautetuksi osoitteeseen http://wadrorstats2017.herokuapp.com/

## Mitä seuraavaksi?

Jos Rails kiinnostaa, kannattaa tutustumista jatkaa esim. seuraaviin suuntiin

* https://leanpub.com/tr5w The Kirja. Kannattaa __ehdottomasti__ hankkia ja lukea kannesta kanteen
* http://guides.rubyonrails.org/ Paljon hyvää asiaa...
* http://railscasts.com/ erinomaisia yhteen teemaan keskittyviä videoita. Uusia videoita ei valitettavasti ole tullut yli vuoteen, toivottavasti sivu aktivoituu vielä. Useimmat maksulliset pro-episodit näköjään löytyvät youtubesta...
* https://www.ruby-toolbox.com/ apua gemien etsimiseen
* [Eloquent Ruby](http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104) erinomainen kirja Rubystä.

Ja jos aikaa ja kiinnostusta riittää, kannattaa osallistua periodilla 4 pidettävään Web-palvelinohjelmointi Ruby on Rails
[harjoitustyöhön](https://github.com/mluukkai/WebPalvelinohjelmointi2017/wiki/harjoitustyö)
