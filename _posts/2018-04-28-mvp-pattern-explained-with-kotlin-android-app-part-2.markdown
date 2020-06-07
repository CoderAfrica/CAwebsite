---
layout: post
title:  "MVP Pattern explained with a Kotlin Android App (Part 2)!"
date:   2018-04-28 21:39:49 +0100
categories: kotlin
---

Building a Kotlin App with the MVP Pattern.

In the last article, I introduced the MVP Pattern. Here, I will jump  straight to coding to show how it is setup. The complete source code for this project is available on [Github](https://github.com/CoderAfrica/BibleApp).

**Our App Context**

To show MVP in a real world app, we will be building a bible app. It will  be an offline KJV app. UI will be 3 simple drop-down menus(Spinner).

One for the Bible Book, another for the bible chapter and the last for the bible verse.

Then a scrollable view(Recyclerview) under the drop-down menus that displays the book.

The result will not be the best looking app you know. But since this was a rushed work to explain this concept, bear with me.

![img](https://miro.medium.com/max/34/1*0kqrN0WnbkYhAbiL1zXlqw.png?q=20)

![img](https://miro.medium.com/max/1080/1*0kqrN0WnbkYhAbiL1zXlqw.png)

**App Setup**

I found a KJV bible file online and wrote some code to transform it into a queryable format. So I have 3 file in my assets folder —  biblebooks.json, bibleChapter.json and bibleverse.json.

All are 3 JSON Arrays with the following objects in this type of schema.

```
BibleBooks - {  "abbrev": "gn",  "bookname": "Genesis",  "noOfChapter": 50,  "bookid": 1}
```

BibleChapters

```
BibleChapters - {"chapterid": 1,"noOfVerses": 31,"bookid": 1},
```

BibleVerses –

```
{"verseid":22,"chapterid":1,"versetext":"And God gave them his blessing, saying, Be fertile and have increase, making all the waters of the seas full, and let the birds be increased in the earth.","bookid":1}
```

Every Verse contains the bookId and ChapterID. Every Chapter contains the bookId.

**App Dependencies**

This App uses [Dagger2 ](https://google.github.io/dagger/)for dependency Injection, [Realm ](https://realm.io/docs/java/latest/)for Database and [ButterKnife ](http://jakewharton.github.io/butterknife/)for View Injection. Knowledge of these libraries are not required to understand this project.

**Now to MVP**

In MVP, the most important part is the MVP Contract. Here you state and  define the all your data models and how they would relate to the view.

Here is the base MVP contract inherited by every View (activities or fragment) in our android application.

```kotlin
interface MVPContract {    
    interface View     
    interface Presenter<V : View> {        
        fun getView(): V?        
        fun attachView(view: V)        
        fun detachView()    
    }    
    interface Component<V : View, out P : Presenter<V>> { 
               fun presenter(): P 
        }
    }
```

We have a View (Activity or Fragment). They can contain any native view  (TextView, ViewPager, NavigationView) etc. we don’t care what they  contain.

We have the Presenter for every View. Every Presenter is instantiated with the View Object it is presenting. It has 3 core functions. Attach the  view, detach it and return a reference to it.

Concrete implementations can add more functionalities depending the native views the activity or fragment contain and the user interactions with it.

We have the Component. This is used by Dagger 2. I never write any app  without dependency injection. It injects the Presenter into the  View(Activity/Fragment). I wanted to do without it here but that part of me wouldn’t stop.

Also, For Code reuse, I decided to abstract Base Abstract classes for Individual Activities and Fragments.

Here they are.

```kotlin
abstract class MVPDaggerRealmActivity<V : MVPContract.View, 
out P : MVPContract.Presenter<V>,       
out C : MVPContract.Component<V, P>> 
: AppCompatActivity(), MVPContract.View {  
    
      protected val presenter: P by lazy { component.presenter() }  
      protected val component: C by lazy { createComponent() }    
      
      lateinit var realm: Realm    
      protected abstract fun createComponent(): C    
      @Suppress("UNCHECKED_CAST")    
      override fun onCreate(savedInstanceState: Bundle?) {   
               super.onCreate(savedInstanceState)      
               presenter.attachView(this as V)     
               realm = Realm.getDefaultInstance()   
       }   
      override fun onDestroy() { 
            super.onDestroy()      
            presenter.detachView()
            realm.close() 
          }
    }
```

The Fragment MVP base class also looks like the activitiy except that it extends

```kotlin
: Fragment(), MVPContract.View
```

Instead of AppCompatActivity.

The MVPDaggerRealmActivity extends AppCompatActivity and Implements  MVPContact.View. The views and view interactions will all be defined in  the View.

It has 2 protected fields.

**1** the presenter that will be injected by the dagger 2 component.

**2** the dagger 2 component that will be created in the concrete class that  extends this class by providing the CreateComponent function  implementation.

**Finally, our business Logic**

Skipping the MainActivity and jumping to the BibleFragment.

Here is our BibleContract.

```kotlin
interface BibleContract {  
      interface View : MVPContract.View {       
           fun showInitialView(bookId : Int, chapterId: Int)
           fun getChapterByBook(bookId: Int?,chaptersInBook:RealmResults<BibleChapter>)   

           fun getVersesByChapter(bookId: Int?, chapterId: Int?, versesInChapter : RealmResults<BibleVerses>)    
        }    
       interface Presenter<V : View> : MVPContract.Presenter<V> {       
           val bibleBooks: RealmResults<BibleBooks>       
           fun getBibleChaptersByBookId(bookId: Int?): RealmResults<BibleChapter>        fun getBibleVerseByBookIdAndChapterId(bookId: Int, chapterId: Int): RealmResults<BibleVerses>   
         }    
       interface Component<V : View, out P : Presenter<V>> : MVPContract.Component<V, P>
     }
```

This defines the contract between our BibleView and Model via the Bible Presenter.

We can see that our bible View will show an initial view. It will can also get chapters by bookId and verses by chapterId. They are all defined  here because they are the basic things the view is expected to do.

We will be using spinners on 1 fragment (one screen). We can decide  tomorrow to have a new fancy UI that will involve 3 screens and many  buttons clicks and animations. No matter what we do, we will still need  those 3 functions on the view because at the end of the day, we must  show an initial view, display chapters in a selected view and display  the verses in a particular selected verse.

**Our Presenter**

```kotlin
class BiblePresenter 
@javax.inject.Inject constructor(var context: Context, var realm: Realm)    : MVPPresenter<BibleContract.View>(), BibleContract.Presenter<BibleContract.View> { 
    
       override fun getBibleVerseByBookIdAndChapterId(bookId: Int, chapterId: Int): RealmResults<BibleVerses> = realm.where(BibleVerses::class.java)
                                .equalTo("bookid", bookId)
                                .equalTo("chapterid", chapterId) 
                                .sort("verseid", Sort.ASCENDING)    
                                .findAll() 
       override fun getBibleChaptersByBookId(bookId: Int?): RealmResults<BibleChapter> =                        realm.where(BibleChapter::class.java)    
                                .equalTo("bookid", bookId)     
                                .sort("chapterid", Sort.ASCENDING)      
                                .findAll() 
       override val bibleBooks: RealmResults<BibleBooks> 
              get() = realm.where(BibleBooks::class.java).findAll()
        }
```

Our BiblePresenter implement the MVPPresenter in the BibleContract Class.  It supplies the List of BibleBooks, List of BibleChapters filtered by  the bookId and the list of bible verses in the chapter.

This is the where all data collection, transformation and interaction with the database takes place.

**Finally, Our View (Bible Fragment)**

```kotlin
class BibleFragment : MVPDaggerFragment<BibleContract.View, BiblePresenter, BibleComponent>(), BibleContract.View {
```

Our BibleFragment implements the MVPDaggerFragment and implements the BibleContract.View.

```kotlin
override fun createComponent(): BibleComponent =
         DaggerBibleComponent.builder()   
            .appComponent(App.component) 
            .biblePresenterModule(BiblePresenterModule(activity!!, (activity as MainActivity).realm))
            .build()
            
     @BindView(R.id.bible_book_spinner)@JvmFieldvar bibleBooksSpinner: Spinner? = null
```

Here is dagger doing the BibleComponent Instantiation and ButterKnife injecting the views into the fragments.

Like I said before, android Native view determine how they accept their data. The Spinner accept data via an Array Adapter.

To get the data for our bible spinner, we call the presenter and create a spinner array adapter with it.

```kotlin
val bibleBooks = presenter.bibleBooksval adapter = ArrayAdapter<BibleBooks>(activity, android.R.layout.simple_spinner_item, bibleBooks)

adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)bibleBooksSpinner?.adapter = adapter
```

Our fragment implements the BibleContract.View interface and must implement it functions.

In the case of the current UI we are using, 3 spinners and a Recyclerview that displays the verse text.

**InitialView**

```kotlin
override fun showInitialView(bookId : Int, chapterId: Int) { 
       val result = presenter.getBibleVerseByBookIdAndChapterId(bookId, chapterId) 
    
       bibleReadAdapter = BibleAdapter(activity as MainActivity, result, 1)    
       
       readBibleRecyclerView!!.adapter = bibleReadAdapter}
```

In our initialview, we set bookId and chapterId to 1. when the app is  first launched, it loads Genesis 1. All the verses in the book is gotten from the presenter getBibleVerseByBookIdAndChapterId function.

```kotlin
val result = presenter.getBibleVerseByBookIdAndChapterId(bookId, chapterId)
```

we set the verses list into the bible adapter and set the recyclerview adapter to the bible adapter

```kotlin
bibleReadAdapter = BibleAdapter(activity as MainActivity, result, 1)readBibleRecyclerView!!.adapter = bibleReadAdapter
```

**GetChapterByBookId and GetVersesByChapterId**

```kotlin
override fun getChapterByBook(bookId: Int?, chaptersInBook: RealmResults<BibleChapter>) {    
    val adapter = ArrayAdapter<BibleChapter>(activity, android.R.layout.simple_spinner_item, chaptersInBook)adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
    chapterSpinner!!.adapter = adapter   
    chapterSpinner!!.onItemSelectedListener = object : AdapterView.OnItemSelectedListener {     
           override fun onItemSelected(adapterView: AdapterView<*>, view: View, position: Int, l: Long) {  
            if (position >= 0) {    
                val chapterId = chaptersInBook[position]?.chapterid   
                val result = displayVerseAndRespondToClicks(bookId, chapterId, 1)  
                
                 getVersesByChapter(bookId, chapterId, result) 
               }      
          } 
       override fun onNothingSelected(adapterView: AdapterView<*>){        } 
       
          }  
      chapterSpinner!!.setSelection(0)
      
      }
```

GetChapterByBookId is used by the bible spinner. An arrayAdapter is created with a dropdown view

```kotlin
val adapter = ArrayAdapter<BibleChapter>(activity, android.R.layout.simple_spinner_item, chaptersInBook)

adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
```

when the dropdown menu is clicked, we get the bookId for the book clicked  and the chapter id for the current chapter selected. We then get all the verses that belong to the chapters and set it to start from the  beginning.

```kotlin
val chapterId = chaptersInBook[position]?.chapteridval result = displayVerseAndRespondToClicks(bookId, chapterId, 1) //Start from the first 

versegetVersesByChapter(bookId, chapterId, result)chapterSpinner!!.setSelection(0)
```

The chapterId dropdown is similar to the bookId dropdown.

When the verseId dropdown is clicked, it just scrolls to position.

```kotlin
readBibleRecyclerView?.scrollToPosition(verseId)
```

**Conclusion.**

We can see how this pattern ease our development and decouples the views  from the UI. The contract knows nothing about the native views (3  spinners and recyclerview). It only deals with what the view will need  to get from the model and the presenter is available to deliver the data to it.

We can refactor our UI a million times. The only thing that will change is our Fragment and maybe BibleContract. View depending on the native  views we use and how complex we design our app flow. We will not have to rewrite the views every time we are using a new views with different  data access mechanisms.

I hope you enjoyed the article.