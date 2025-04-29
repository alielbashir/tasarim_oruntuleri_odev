# Restoran Rezervasyon ve Siparis Sistemi

**Hazırlayan:** Ali Elbashir

## Senaryo

Bu projede bir restoranın rezervasyon, sipariş, menü oluşturma, şikayet yönetimi ve müşteri bilgilendirme süreçlerini kapsayan bir sistem tasarlanmıştır. Sistemin amacı, restoran içinde yürütülen operasyonları daha düzenli ve sürdürülebilir hale getirmek, farklı aktörler arasındaki iletişimi standartlaştırmak ve tekrarlayan işlemleri optimize etmektir. Çözümde farklı tasarım örüntüleri kullanılarak sistemin genişleyebilir, modüler ve bakımı kolay olması hedeflenmiştir.

## Kullanılan Tasarım Örüntüleri

| Tasarım Örüntüsü        | Sınıf Adı            | Açıklama |
|-------------------------|----------------------|----------|
| Singleton               | ReservationManager   | Rezervasyon işlemlerinin uygulama boyunca merkezi ve tek bir örnek tarafından yönetilmesi sağlanır. |
| Factory                 | MealFactory          | Tek tek yemek türlerinin (`MainCourse`, `Dessert`, `Drink`) `type` parametresiyle oluşturulması için kullanılır. |
| Abstract Factory        | MenuFactory          | Birbiriyle ilişkili yemek gruplarını içeren menü kombinasyonlarını (örneğin çocuk veya vegan menü) tek seferde üretmek için kullanılır. |
| Builder                 | CustomMenuBuilder    | Menü öğelerinin (başlangıç, ana yemek, tatlı) kullanıcı isteğine göre adım adım bir araya getirilmesini sağlar. |
| Prototype               | Order                | Daha önce oluşturulmuş bir siparişin birebir kopyalanarak yeniden kullanılmasına olanak tanır. |
| Mediator                | OrderCenter          | Farklı birimler (müşteri, garson, mutfak, kasa) arasında doğrudan bağlantı olmadan mesajlaşma sağlar. |
| Chain of Responsibility | ComplaintHandler     | Gelen şikayetlerin sıralı bir şekilde ilgili kişilere yönlendirilmesini ve uygun olanın işlemesini sağlar. |
| Observer                | NotificationService  | Müşteriler sisteme abone olur ve sistemdeki olaylardan (örneğin sipariş durumu) otomatik olarak haberdar edilir. |

## Genel Sistem Diyagramı

```mermaid
classDiagram


class ReservationManager["ReservationManager(Singleton)"] {
    +makeReservation()
    +cancelReservation()
}

class OrderCenter["OrderCenter(Mediator)"] {
    +registerColleague()
    +sendMessage()
}

class NotificationService["NotificationService(Observer)"] {
    +subscribe()
    +unsubscribe()
    +notify()
}

class ComplaintHandler["ComplaintHandler(Chain of Responsibility)"] {
    +handleComplaint(Complaint)
    +setNext(handler)
}

class Order["Order(Prototype)"] {
    +clone()
    +addMeal()
    +getTotalPrice()
}

class CustomMenuBuilder["CustomMenuBuilder(Builder)"] {
    +reset()
    +addStarter()
    +addMainCourse()
    +addDessert()
    +getMenu()
}

class MenuFactory["MenuFactory(Abstract Factory)"] {
    +createVeganMenu()
    +createKidsMenu()
}

class MealFactory["MealFactory(Factory)"] {
    +createMeal(type)
}


class Customer {
    +makeReservation()
    +placeOrder()
}

class Waiter {
    +takeOrder()
    +deliverOrder()
}

class Kitchen {
    +prepareMeal()
}

class Cashier {
    +processPayment()
}

class Menu {
    +addMeal()
    +getMeals()
}

class Meal {
    +getDescription()
    +getPrice()
}

class MainCourse
class Dessert
class Drink
class Complaint {
    -description
    -severity
}


Meal <|-- MainCourse
Meal <|-- Dessert
Meal <|-- Drink


Customer --> ReservationManager : uses
Customer --> OrderCenter : communicates
Waiter --> OrderCenter : communicates
Kitchen --> OrderCenter : communicates
Cashier --> OrderCenter : communicates

Customer --> NotificationService : subscribes
NotificationService --> Customer : notifies

Customer --> CustomMenuBuilder : build custom menu
CustomMenuBuilder --> Menu : builds
MenuFactory --> CustomMenuBuilder : provides structure

Menu --> MainCourse : includes
Menu --> Dessert : includes
Menu --> Drink : includes

MealFactory --> Meal : creates
Order --> Meal : contains
Customer --> Order : places

Customer --> ComplaintHandler : files
ComplaintHandler --> ComplaintHandler : forwards
ComplaintHandler --> Complaint : handles
```

## Tasarım Örüntülerine Ait UML Diyagramları

### Singleton – ReservationManager
```mermaid
classDiagram
class ReservationManager {
    -static instance : ReservationManager
    -ReservationManager()
    +getInstance() ReservationManager
    +makeReservation()
    +cancelReservation()
}
```

### Factory – MealFactory
```mermaid
classDiagram
class MealFactory {
    +createMeal(type: String) Meal
}
class Meal {
    +getDescription() String
    +getPrice() float
}
class MainCourse
class Dessert
class Drink
Meal <|-- MainCourse
Meal <|-- Dessert
Meal <|-- Drink
```

### Abstract Factory – MenuFactory
```mermaid
classDiagram
class MenuFactory {
    +createVeganMenu() Menu
    +createKidsMenu() Menu
}
class CustomMenuBuilder
MenuFactory --> CustomMenuBuilder : provides menu data
```

### Builder – CustomMenuBuilder
```mermaid
classDiagram
class CustomMenuBuilder {
    +reset()
    +addStarter(item: Starter)
    +addMainCourse(item: MainCourse)
    +addDessert(item: Dessert)
    +getMenu() Menu
}
class Menu {
    +addMeal()
    +getMeals()
}
class Starter
class MainCourse
class Dessert
Menu --> Starter
Menu --> MainCourse
Menu --> Dessert
CustomMenuBuilder --> Menu
```

### Prototype – Order
```mermaid
classDiagram
class Order {
    +clone() Order
    +addMeal(meal: Meal)
    +getTotalPrice() float
}
class Meal
Order --> Meal
```

### Mediator – OrderCenter
```mermaid
classDiagram
class OrderCenter {
    +registerColleague(colleague: Colleague)
    +sendMessage(sender: Colleague, receiver: Colleague, message: String)
}
interface Colleague
class Customer
class Waiter
class Kitchen
class Cashier
Customer ..|> Colleague
Waiter ..|> Colleague
Kitchen ..|> Colleague
Cashier ..|> Colleague
```

### Chain of Responsibility – ComplaintHandler
```mermaid
classDiagram
class ComplaintHandler {
    +handleComplaint(complaint: Complaint)
    +setNext(handler: ComplaintHandler)
}
class Complaint {
    +description: String
    +severity: int
}
ComplaintHandler --> ComplaintHandler : forwards
ComplaintHandler --> Complaint : handles
```

### Observer – NotificationService
```mermaid
classDiagram
class NotificationService {
    +subscribe(customer: Customer)
    +unsubscribe(customer: Customer)
    +notify(event: String)
}
class Customer
NotificationService --> Customer
Customer --> NotificationService
```

