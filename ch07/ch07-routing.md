# Chapter 7 Routing

This chapter introduces the Angular routing mechanism. The app now has two views: the existing heros view and a dashboard view. The hero detail views is used as a child in both top level views.

## 1 Add Routing Module

All Angular functions are provided via a module. The Angular routing functions of an application are usually defined in one or more so-called routing modules.

Use the CLI command to generate the root routing module: `ng generate module app-routing --flat --module=app`. The `--flat` option tells the CLI to generate the module file in the `src/app` folder, not in its own folder. The `--module=app` register the new routing module in the `AppModule`. You can use any valid module name for the routing module. However, `app-routing` makes it clear that this is the root level module. The generate file `src/app/app-routing.ts` has the following content:

```ts
import { NgModule } from '@angular/core'
import { CommonModule } from '@angular/common'

@NgModule({
  declarations: [],
  imports: [CommonModule],
})
export class AppRoutingModule {}
```

The `CommonModule` has common directives such as `nfIf`, `ngFor` etc. used by most components. However, you shouldn't define any components in a routing moudle. Therefore you can delete the `imports` and `declarations` in the `NgModule` decorator.

The routing module uses the `Routes` type defined in Angular `RouterModule`. Also, to make the router directives available to all components, the routing module exports the `RouterModule`. The changed code look like the following:

```ts
import { NgModule } from '@angular/core'
import { Routes, RouterModule } from '@angular/router'

@NgModule({
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

## 2 Add Routes

There is a router service provided by the `RouterModule` that controls all route navigation. Routes tell the router service which view (consisting of one or more components) to display when the url changes, often as a result of clicking a link or manual change of the url in the browser address bar.

A route is defined by a path and a component. The `Routes` type is an array of routes. The heros component can be defined in this route: `{ path: 'heroes', component: HeroesComponent }`. The path in url is `/heroes` and the component to display is the`HeroesComponent`. Edit the `src/app/app-routing.ts` to have the following content:

```ts
import { NgModule } from '@angular/core'
import { Routes, RouterModule } from '@angular/router'
import { HeroesComponent } from './heroes/heroes.component'

const routes: Routes = [{ path: 'heroes', component: HeroesComponent }]

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

You should call `RouterModule.forRoot(routes)` for the applicaton root level routes. In one application, it can be called once in the root module. You can find it is in the array of the `imports` property of the `@NgModule` decorator of the `AppModule`.

## 3 Navigate to Heroes

To display a component when its path matches in an url, use the `router-outlet` element tag. It is provided by the `RouterModule`. Change the `src/app/app.component.html` to have the following content:

```html
<h1>{{ title }}</h1>
<nav><a routerLink="/heroes">Heroes</a></nav>
<router-outlet></router-outlet>
```

Now the home page `/` only has a title. When you change the url to `/heroes`, you can see the list of heroes.

You can click the link to navigate to the heroes view too. The `routerLink` attribute, again a directive provided by the `RouterModule`, sets the link url to its value when clicked.

## 4 Add a Dashboard View

Now add a new component `DshboardComponent` that display 4 heroes from the heroes list. Use `ng g c dashboard` to create a new dashboard component. The template file has the following content:

```html
<h3>Top Heroes</h3>
<ul>
  <li *ngFor="let hero of heroes"><a routerLink="/detail/{{hero.id}}">{{ hero.name }}</a></li>
</ul>
```

The dashboard component class has the following code:

```ts
import { Component, OnInit } from '@angular/core'
import { Hero } from '../hero'
import { HeroService } from '../hero.service'
import { Observable } from 'rxjs'

@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html',
  styleUrls: ['./dashboard.component.css'],
})
export class DashboardComponent implements OnInit {
  heroes: Hero[]

  constructor(private heroService: HeroService) {}

  ngOnInit() {
    const heroes$: Observable<Hero[]> = this.heroService.getHeroes()
    heroes$.subscribe(hs => (this.heroes = hs.slice(1, 5)))
  }
}
```

The `dashboard.component.css` file has the following content:

```css
/* DashboardComponent's private CSS styles */
[class*='col-'] {
  float: left;
  padding-right: 20px;
  padding-bottom: 20px;
}
[class*='col-']:last-of-type {
  padding-right: 0;
}
a {
  text-decoration: none;
}
*,
*:after,
*:before {
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
}
h3 {
  text-align: center;
  margin-bottom: 0;
}
h4 {
  position: relative;
}
.grid {
  margin: 0;
}
.col-1-4 {
  width: 25%;
}
.module {
  padding: 20px;
  text-align: center;
  color: #eee;
  max-height: 120px;
  min-width: 120px;
  background-color: #3f525c;
  border-radius: 2px;
}
.module:hover {
  background-color: #eee;
  cursor: pointer;
  color: #607d8b;
}
.grid-pad {
  padding: 10px 0;
}
.grid-pad > [class*='col-']:last-of-type {
  padding-right: 20px;
}
@media (max-width: 600px) {
  .module {
    font-size: 10px;
    max-height: 75px;
  }
}
@media (max-width: 1024px) {
  .grid {
    margin: 0;
  }
  .module {
    min-width: 60px;
  }
}
```

Now add a dashboard link to the shell component template, the `src/app/app.component.html` file has the following code:

```html
<h1>{{ title }}</h1>
<nav>
  <a routerLink="/dashboard">Dashboard</a>
  <a routerLink="/heroes">Heroes</a>
</nav>
<router-outlet></router-outlet>
```

You also want to make it as the default route.

Now the routes has the following content:

```ts
const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard', component: DashboardComponent },
  { path: 'heroes', component: HeroesComponent },
]
```

The first path is the default path that matches `/`, the root url path. The `pathMatch: 'full'` means that it cannot have anything after the `'/'` because by default the empty string `''` matches any path as its prefix or suffix.

## 5 Navigating to Details

Currently the deatil component is at the page bottom. It should be accessible via three methods:

- By clicking a hero in the dashboard
- By clicking a hero in the heroes list
- By manually typing a "deep link" URL in the browser address bar.

First, remove `<app-hero-detail>` element from the `heroes.component.html`. Change `src/app/heroes/heroes.component.html` as the following:

```html
<h2>My Heroes</h2>
<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <a routerLink="/detail/{{ hero.id }}"> {{ hero.id }} {{ hero.name }}</a>
  </li>
</ul>
```

Also remove `onSelect()` and `selectedHero` from the heroes component class file. The class has the following content:

```ts
import { Component, OnInit } from '@angular/core'
import { Hero } from '../hero'
import { HeroService } from '../hero.service'
import { Observable } from 'rxjs'

@Component({
  selector: 'app-heroes',
  templateUrl: './heroes.component.html',
  styleUrls: ['./heroes.component.css'],
})
export class HeroesComponent implements OnInit {
  heroes: Hero[]

  constructor(private heroService: HeroService) {}

  ngOnInit() {
    const heroes$: Observable<Hero[]> = this.heroService.getHeroes()
    heroes$.subscribe(hs => (this.heroes = hs))
  }
}
```

Finally, add a hero detail route in `src/app/app-routing.module.ts`.

```ts
import { HeroDetailComponent }  from './hero-detail/hero-detail.component'

{ path: 'detail/:id', component: HeroDetailComponent },
```

## 6 Get Router Parameter

So far the hero id is embedded in the `routeLink`. A user may also input a url like `~/detail/11` in the browser's address bar. In `HeroDetailComponent`, you need to retrieve the hero id from the url using the following code:

```ts
import { Component, OnInit, Input } from '@angular/core'
import { ActivatedRoute } from '@angular/router'

import { Hero } from '../hero'
import { HeroService } from '../hero.service'

@Component({
  selector: 'app-hero-detail',
  templateUrl: './hero-detail.component.html',
  styleUrls: ['./hero-detail.component.css'],
})
export class HeroDetailComponent implements OnInit {
  hero: Hero
  ngOnInit(): void {
    this.getHero()
  }

  constructor(private route: ActivatedRoute, private heroService: HeroService) {}

  getHero(): void {
    // The JavaScript (+) operator converts the string to a number, which is what a hero id should be.
    const id = +this.route.snapshot.paramMap.get('id')
    this.heroService.getHero(id).subscribe(hero => (this.hero = hero))
  }
}
```

Open HeroService and add this getHero() method:

```ts
getHero(id: number): Observable<Hero> {
  return of(HEROES.find(hero => hero.id === id))
}
```

## 7 Find a Way Back

Add a back button in hero deatils to let user go back.

Add the following to the deatils html file:

```html
<button (click)="goBack()">go back</button>
```

Add the following code to the component class file:

```ts
goBack(): void {
  this.location.back();
}
```

After the changes, the `src/app/hero-detail/hero-detail.component.ts` has the following content:

```ts
import { Component, OnInit } from '@angular/core'
import { Hero } from '../hero'
import { ActivatedRoute } from '@Angular/router'
import { HeroService } from '../hero.service'
import { Location } from '@angular/common'

@Component({
  selector: 'app-hero-detail',
  templateUrl: './hero-detail.component.html',
  styleUrls: ['./hero-detail.component.css'],
})
export class HeroDetailComponent implements OnInit {
  hero: Hero

  constructor(
    private route: ActivatedRoute,
    private heroService: HeroService,
    private location: Location,
  ) {}

  ngOnInit() {
    this.getHero()
  }

  private getHero() {
    const id = this.route.snapshot.paramMap.get('id')
    const getHero$ = this.heroService.getHero(Number(id))
    getHero$.subscribe(hero => (this.hero = hero))
  }

  goBack(): void {
    this.location.back()
  }
}
```
