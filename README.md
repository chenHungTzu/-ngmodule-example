# 建立 Angular 模組


## 摘要
> * 使用 ng-packagr 建立模組
> * 使用 @angular-devkit/build-ng-packagr 建立模組
> * 如何使用
    * Routing
    * Component 
    * NgStore
    * Service



## 使用 ng-packagr 建立模組

1 建立一個空的angular專案

```
ng new 專案名稱
```

2 下載封裝模組

```
npm install ng-packagr

```

3 建立 `public_api.ts`  於專案目錄 `./src/` 中

4 建立 `ng-package.json`  於專案目錄 `./src/` 中 , 內容大致如下 ：

```javascript
  {
        "$schema": "./node_modules/ng-packagr/ng-package.schema.json",
        "lib": {
            "entryFile": "./src/public_api.ts"
        },
        "whitelistedNonPeerDependencies": [
        "."
        ]
  }
```
5  修改檔案 `./package.json` , 內容大致如下 ：

```javascript
  {
   "name": "ptc-common-module",
   "version": "0.0.7",
   "scripts": {
    ...
    "packagr": "ng-packagr -p ng-package.json"  // << add line
  },
  ...
```
6 將即將公布的類別 export 至 `./src/public_api.ts` 中 , 內容範例如下 ：

<img width="650" height="400" alt="script" src="https://raw.githubusercontent.com/chenHungTzu/ng-module-example/master/api.png">

7 執行封裝行為 , 對應結果將會產生至 `./dist` 目錄下

```
npm run packagr
```

8 進行發布行為 , cd 至 `./dist` 內執行指令

```
npm publish
```


## 使用 @angular-devkit/build-ng-packagr 建立模組

> `＠angualr/cli` 版本需在`6+`



1 建立一個空的angular專案

```
ng new 專案名稱
```
2 建立模組

```
ng g library 模組名稱
```
執行完畢後 , 將會建立 `./projects` 目錄 內容如下 ：

> 模組名稱 ： shared

<img width="300" height="400" alt="script" src="https://raw.githubusercontent.com/chenHungTzu/ng-module-example/master/nglib.png">

在這邊你會發現 , 相關的發布定義檔 `ng-package.json` , `public_api.ts` 都已經替你建立好了 ,在 `./angular.json` 中 , 也會發現相關 build script 也會一併建立 , 如圖 ：


<img width="500" height="400" alt="script" src="https://raw.githubusercontent.com/chenHungTzu/ng-module-example/master/script.png">

3 下載打包模組 
> 空的 angular 專案不包含以下模組 , 需要自行下載

```
npm install @angular-devkit/build-ng-packagr ng-packagr tsickle
```

4 執行封裝行為 , 對應結果將會產生至 `./dist` 目錄下

```
ng build
```

5 進行發布行為 , cd 至 `./dist` 內執行指令

```
npm publish
```

## 如何使用

現在我們已經有能力將 angular 相關程式碼獨立成一個函式庫 , 以下有幾個範例將展示如何運用既有的封裝 , 以便在後續專案能夠減少重工的機會。

*  [介入 ngRouting](#介入-ngRouting)

*  [重用/複寫 service](#重用/複寫-service) 

*  [介入 ngStore / Action](#介入-ngStore-/-Action)  

*  [重用 component](#重用-component)


### 介入 ngRouting

> [情境] 目前已經有既有的模組,名稱為<span style="color:red">pphone</span> , routing 與對應component 已經準備好 , 程式碼如下 :

> telephone-routing.module.ts : 

> ```javascript
> import { NgModule } from '@angular/core';
> import { Routes, RouterModule } from '@angular/router';
> import { TelephoneComponent } from './components/telephone/telephone.component';
> import { ErrorComponent } from './components/error/error.component';
> const routes: Routes = [
>  {path : '' , component : TelephoneComponent},
>  {path : '**' , component : ErrorComponent},
> ];
> @NgModule({
>  imports: [RouterModule.forChild(routes)],
>  exports: [RouterModule]
> })
> export class TelephoneRoutingModule { }
> ```

> telephone.module.ts :
> ```javascript
> @NgModule({
>  declarations: [
>    ControlPanelComponent,
>    CommunicationRecordsComponent,
>    TelephoneComponent,
>    ErrorComponent
>  ],
>  providers: [
>    ...
>  ],
>  imports: [
>	...
>    CommonModule,
>    FormsModule,
>    TelephoneRoutingModule, 
>  ],
>  exports: [
>   TelephoneRoutingModule,  
>    ControlPanelComponent,
>    CommunicationRecordsComponent,
>    TelephoneComponent,
>    ErrorComponent
>  ]
> })
> export class TelephoneModule { }
> ```

> 接下來我們將從 npm 下載該模組進行運用


1 建立一個空的angular專案

```
ng new 專案名稱
```

2 下載模組

```
npm install pphone
```

3 下載完畢後 , 將自動建立目錄 `./node_module/pphone`

4 於空專案內建立 sub module  , 請參閱 ：[ng generate](https://angular.io/cli/generate#module-command) 

```
ng g module phone
```
執行完畢後，將會於 `./src/app/` 建立目錄 `phone` 

5 修改 `./src/app/phone/phone.module.ts` 

```javascript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TelephoneModule} from 'pphone';


@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    TelephoneModule, // add this line.
  ],
  
})
export class PhoneModule { }
```

6 修改 `./src/app/app-routing.module.ts`  , 請參閱 ：[ng route](https://angular.io/tutorial/toh-pt5) 


```javascript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

const routes: Routes = [
  { path: 'telephone', loadChildren: './phone/phone.module#PhoneModule' }, // << add this line .
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
7 展示結果如下 ：

![demo](https://github.com/chenHungTzu/ng-module-example/blob/master/demo.gif?raw=true)


### 重用/複寫 service
> [情境] 目前已經有既有的模組,名稱為<span style="color:red">pcommon </span> , 共用 component 已經準備好 , 程式碼如下 :
> 
> http.service.ts
> 
> ```
> @Injectable()
> export class HttpService {
>  constructor(public http: HttpClient,
>              public config: HttpConfig) { }
>
>  get<TResult>(params: HttpParameter): Observable<TResult> {
>    return this.http.get<TResult>(params.hostUrl, params)
>      .pipe(
>        timeout(this.config.MAX_TIME_OUT),
>        catchError(this.errorHandler),
>        retry(this.config.RETRY_COUNT)
>      )
>  }
>  post<TResult>(params : HttpParameter) : Observable<TResult>{
>    return this.http.post<TResult>(params.hostUrl , params.body , params)
>            .pipe(
>              timeout(this.config.MAX_TIME_OUT),
>              catchError(this.errorHandler),
>              retry(this.config.RETRY_COUNT)
>    )
>  }
> 
>  errorHandler(error: HttpErrorResponse) {
>    /**
>     * DO SOMETHING
>     */
>    console.log(error)
>    return throwError(error);
>  }
> }
> ```

> http-parameter.model.ts

> ```javascript
> import { HttpParams, HttpHeaders } from '@angular/common/http'
> export class IHttpOptions
> { 
>    headers?: HttpHeaders | {
>        [header: string]: string | string[];
>    };
>    observe?: 'body';
>    params?: HttpParams | {
>        [param: string]: string | string[];
>    };
>    reportProgress?: boolean;
>    responseType? : 'json'
>    withCredentials?: boolean;
> }
> export class HttpParameter extends IHttpOptions {
>
>    hostUrl : string
>    body? : any = {}
> }
> ```

1 建立一個空的angular專案

```
ng new 專案名稱
```

2 下載模組

```
npm install pcommon
```

3 下載完畢後 , 將自動建立目錄 `./node_module/pcommon `


4 修改 `./app.module.ts`

```javascript 

...
import { PtcCommonModule } from 'pcommon';
import { environment } from '../environments/environment';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    PtcCommonModule,   // << add this line.
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }


```

5 引用方式 , ex : `./service/AuthenticationService.ts`

```javascript
 
...
import { HttpService } from 'pcommon';
import { HttpResponseModel } from 'pcommon';
import { Observable } from 'rxjs';


@Injectable()
export class AuthenticationService {

    constructor(@Optional() public http: HttpService) {

    }

    /**
     * 登入
     * @param model 
     */
    login(model: MemberModel): Observable<HttpResponseModel<any>> {

        return this.http.post<HttpResponseModel<any>>(
            {
                hostUrl: 'http://{yourip}/Api/Telephone/Login',
                responseType: 'json',
                body: model
            })
    }

    /**
     * 登出
     * @param model 
     */
    logoff(model: MemberModel): Observable<HttpResponseModel<boolean>> {

        return this.http.post<HttpResponseModel<boolean>>(
            {
                hostUrl: 'http://{yourip}/Api/Telephone/LogOff',
                responseType: 'json',
                body: model
            })
    }
}


```

7 複寫 service 
 > 透過angular DI 注射 , 可以讓我們彈性的選擇想要的類別進行註冊 , 如下 ：
 
 ```javascript
 
 import { NgModule } from '@angular/core';
 import { CommonModule } from '@angular/common';
 import { TelephoneModule, HookService, AuthenticationService } from 'pphone';
 import { Auth_Main } from 'src/service/auth.service';
 import { Hook_Main } from 'src/service/hook.service';

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    TelephoneModule,
  ],
  providers: [
    {
    
      // HookService 為封裝層 , Hook_Main 為當前專案中的某個類別
      // 透過 Hook_Main 繼承 HookService , 可複寫封裝內方法 
      
      provide : HookService , useClass : Hook_Main
    },
    {
      provide : AuthenticationService ,useClass : Auth_Main
    }
  ],
})
export class PhoneModule { }
 
 ```
 
 HookService.ts (封裝後)
 
 ```javascript
 
 export declare class HookService {
    afterLogin(result: any): void;
    beforeLogin(model: MemberModel): Observable<any>;
    afterLogoff(result: any): void;
    beforeLogoff(model: MemberModel): Observable<any>;
}
 
 ```
 
 Hook_Main.ts
 
 ```javascript
 
 @Injectable()
	export class Hook_Main extends HookService {
  
		// 透過繼承關係 , 若封裝中有呼叫到該方法 , 將會執行子類別action實作
		// 若無實作 , 則執行的會是父類別的action    
		
		afterLogoff(result: any) : void{
    	    console.log(result);
    	    console.log('afterLogoff!!!_main')
    	}

    	afterLogin(result: any): void {       
    	    console.log(result);
     	   console.log('afterLogin!!!_main')
    	}       

	}
 
 ```
 
 > 相關文獻請參閱 [angular DI](https://angular.io/guide/hierarchical-dependency-injection) 

### 介入 ngStore / Action

> ngrxStore 建置內容這邊不多贅述 , 請參閱 ：[ngrx.io](https://ngrx.io/guide/store)
> 
> 
> public_api.ts
> 
> ```javascript
>
> export * from "./app/core/ngrx/actions/phone.actions";
> export * from "./app/core/ngrx/reducers/index";
> export * from "./app/core/ngrx/effects/index";
>
> ...
>
> ``` 
>
> 與上個範例相同 , 將從 npm 下載該模組進行運用


1  同上個範例步驟1.2.3 

2 修改 `./app.module.ts`

```javascript

...
import { TelephoneModule } from 'pphone';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    TelephoneModule  //add this line.
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```



2 修改 `./src/app/app.component.ts`

```

...

import { Store, select } from '@ngrx/store';
import { State , Login } from 'pphone'
import { skip } from 'rxjs/operators';

...
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {

  title = 'cust';
  
  // 透過DI 注射封裝store
  constructor(public store: Store<State>) { }

  ngAfterViewInit(): void {

	// 這邊訂閱封裝內的store狀態
    this.store.pipe(
      skip(1),
      select(x => x.phone),
    ).subscribe((x) => {
      console.log('debugger');
      console.log(x);
    })

  }
   
  doLogin() {
    let member = new MemberModel();
    member.extension = '9527';
    member.loginID = 'foo'
    
    // dispatch 封裝內的 action
    this.store.dispatch(new Login(member))
  }

}


```


### 重用 component
> [情境] 目前已經有既有的模組,名稱為<span style="color:red">ptc-commomn-module </span> , 共用 component 已經準備好 , 程式碼如下 :


> local-table.component.html
> ```javascript
> <ng2-smart-table [settings]="settings" [source]="data"></ng2-smart-table>
> 
> ```

> local-table.component.ts
> 
> ```javascript
> import { Component, OnInit, Input } from '@angular/core';
> @Component({
>  ...
> })
> export class LocalTableComponent implements OnInit {
>  @Input() data = [
>    {
>      id: 1,
>      name: "Leanne Graham",
>      username: "Bret",
>      email: "Sincere@april.biz"
>    }
>        // ... list of items      
>  ];
>  @Input() settings  = {
>    columns: {
>      id: {
>        title: 'ID'
>      },
>      name: {
>        title: 'Full Name'
>      },
>      username: {
>        title: 'User Name'
>      },
>      email: {
>        title: 'Email'
>      }
>    }
>
>  }
>  constructor() { }
> }
> 
> ```

> ptc-common.module.ts

> ```
>  @NgModule({
>   declarations: [LocalTableComponent],
>   imports: [
>    CommonModule,
>    Ng2SmartTableModule,  //this example using smart table.
>    HttpClientModule
>  ],
> providers:[
>    ...
>  ],
>  exports :[
>    LocalTableComponent, //add this line.
>  ]
> })
> export class PtcCommonModule { }
> ```

> 與上個範例相同 , 將從 npm 下載該模組進行運用


1  同上個範例步驟1.2.3 

2 修改 `./src/app/app.module.ts`

```javascript

...
import { PtcCommonModule } from 'ptc-common-module';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    PtcCommonModule  //add this line.
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```


3 修改 `./src/app/app.component.ts`

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
   data = [
    {
      id: 111,
      name: "anthony",
      username: "陳宏慈",
      email: "motx2152000@ptc-nec.com.tw"
    },
   
  ];

   settings  = {
    columns: {
      id: {
        title: 'ID'
      },
      name: {
        title: 'Full Name'
      },
      username: {
        title: 'User Name'
      },
      email: {
        title: 'Email'
      }
    }

  }
}

```

4 修改 `./src/app/app.component.html`

```
<app-local-table [settings] = "settings" [data]="data"></app-local-table>

```

5 展示結果如下 ： 

<img width="100%"  alt="demo2" src="https://raw.githubusercontent.com/chenHungTzu/ng-module-example/master/demo1.png">

