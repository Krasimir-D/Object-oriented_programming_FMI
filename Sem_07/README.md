# Изключения (Exceptions)

В една програма могат да възникнат различни типове грешки: 
- **синтактични грешки**, които се откриват по време на компилация;
- **семантични грешки**, които се откриват по време на изпълнение на програмата.

Често срещани проблеми, които трябва да могат да бъдат предвидени и да може да се реагира на тях, са например:
- параметри с невалидни стойности, подадени на дадена функция;
- възникване на грешка по време на изпълнение на функция, в следствие на която функцията връща определен резултат;
- грешен формат на потребителски вход на дадена програма.

#### Възникване на грешка в конструктор

Задачата на всеки конструктор е да създаде валиден обект. В допълнение, всяка публична член-функция, която се изпълнява върху обект, предполага начално валидно състояние и трябва да остави обекта отново във валидно състояние.  

Конструкторът не връща стойност, той не може да върне код за грешка. Как да се реагира в случай на проблем или на невалидни входни данни? Дори да се добави аргумент, който да указва дали всичко е преминало успешно, в крайна сметка обект ще бъде конструиран, дори той да не е валиден.  

Досега в конструкторите на класовете с подобни проблеми се справяхме по няколко начина:
1. **Обектът се маркира като невалиден.** Това означава, че при всяка операция (член-функция на класа), която се прилага над обект от този клас, трябва да се проверява дали се работи с валиден обект.  
**Пример:**
```c++
Rational::Rational(int numerator, int denominator) : numerator(numerator), denominator(denominator)
{}
bool Rational::isValid() const
{
	return denominator != 0;
}

int main()
{
	Rational r(1, 0);
	if (r.isValid())
	{
		// code
	}
}
```

2. **Обектът се поправя локално**, подменя се с валиден.  
**Пример:**
```c++
Rational::Rational(int numerator, int denominator) : numerator(numerator)
{
	if (denominator == 0)
	{
		denominator = 1;
	}
	this->denominator = denominator;
}
```

Дали тези подходи са добри? **Не** :bangbang:  
**Проблемът се „смита под килима“**. Не се сигнализира за него.  
Дали има друг механизъм за справяне в такива ситуации? **Да**.  

**Изключенията** са механизъм за сигнализация за неочаквана (изключителна) ситуация, различна от очаквания, основен ход на събитията. Идеята е да се подаде съобщение за проблем, възникнал по време на изпълнение на програмата, и някой, който може да реши проблема, да поеме отговорност и да възстанови нормалния ход на нещата. С изключенията в езика са свързани три ключови думи: **throw**, **try** и **catch**.

## Хвърляне на изключение

За да бъде сигнализирано за възникването на определена неочаквана ситуация, различна от това, което програмата може да обработи, се използва следната конструкция:  
**throw <израз>;**  
<израз> е съобщението към външния свят и може да бъде от произволен тип. Обикновено това е код на грешка (константа от изброен тип), символен низ с описание на грешката, може да бъде дори обект от потребителски дефиниран тип.  

**Примери:**  
throw INVALID_INDEX;  
throw “Index out of range!”;  
throw InvalidStudentsIDException();

## Прихващане на изключения

Ключовата дума **try** се използва, за да дефинира блок от операции, който ще бъде наблюдаван за възникване на изключения. Всяка операция в рамките на блока се проверява. Ако възникне изключение, то се прихваща само ако **try блокът** е последван от **catch блок** с тип, съответстващ на възникналото изключение. Всеки try блок трябва да има поне един catch блок, а може и да е последвано от няколко такива.  

**Пример:**  
```c++
std::cout << "Before the try block..." << std::endl;
try
{
	std::cout << "Inside the try block, before the throw..." << std::endl;
	throw "Error"; // изключението е от тип символен низ
	std::cout << "Inside the try block, after the throw..." << std::endl;
}
catch (const char* expr) // това, което е обявено като тип в catch блока трябва да съвпада с типа на изключението
{
	std::cerr << "Exception caught: " << expr << std::endl;
}
std::cout << "After the try block, after the throw..." << std::endl;
```

- Ако в тялото на try блока **не възникне изключение**, всички catch блокове се игнорират и изпълнението продължава от първата операция след последния catch блок.
- Ако в тялото на try блока **възникне изключение**, което няма съответстващ catch блок или възникне изключение извън try блок, съдържащата го функция (т.е. функцията, в която е възникнало изключението) прекъсва, при което всички локални променливи се освобождават и програмата се опитва да намери try блок в извикващата функция. Извършва се **изкачване по стека (stack unwinding)**.  

```c++
struct MyObject
{
    MyObject()
    {
        std::cout << "MyObject constructor called" << std::endl;
    }
    ~MyObject()
    {
        std::cout << "MyObject destructor called" << std::endl;
    }
};

int main()
{
    try
    {
        MyObject obj1;
        MyObject obj2;
        throw "An exception occurred";
        MyObject obj3;
    }
    catch (const char* e)
    {
        std::cout << "Exception caught: " << e << std::endl;
    }
}
```

![alt_text](https://github.com/MariaGrozdeva/Object-oriented_programming_FMI/blob/main/Sem_07/images/StackUnwinding.png)

### Изключения в конструкторите

Ако конструктор не успее по някаква причина да създаде валиден обект, например при подадени некоректни стойности за член-данните, може да се хвърли изключение, с което да се сигнализира, че обектът не е създаден успешно.  

При възникване на грешка в конструктор, **деструкторът на класа не се извиква** :bangbang:  
Следователно не може да се разчита на него да освободи външни ресурси, ангажирани преди възникването на грешката. 
**Освобождаването на тези ресурси трябва да се осъществи преди хвърлянето на изключението**. Ако в класа има вградени обекти от други класове, за тях ще бъдат извикани деструктори.
```c++
class Item
{
private:
	char* name;
	char id[6];
	double price;
	
public:
	Item(const char* id, const char* name, double price);
	Item(const Item& other);
	Item& operator=(const Item& other);
	~Item();
	// ...
};

Item::Item(const char* name, const char* id, double price)
{
	if (name == nullptr)
	{
		throw "Invalid name!";
	}
	// ако паметта за името е заделена първо
	this->name = new char[strlen(name) + 1];
	strcpy(this->name, name);

	if (!isValid(id)) 
	{
		// трябва да се освободи паметта за името преди да се хвърли изключение
		delete[] this->name;
		throw "Invalid id!";
	}
	strcpy(this->id, id);

	if (price < 0)
	{
		delete[] this->name;
		throw "Invalid price!";
	}
	this->price = price;
}
```

По- изчистена реализация на същата функция:
```c++
Item::Item(const char* name, const char* id, double newPrice){
	if (!isValid(id))
	{
		throw "Invalid id!";
	}
	strcpy(this->id, id);

	if (price < 0)
	{
		throw "Invalid price!";
	}
	this->price = price;

	// може името да се обработи последно 
	if (name == nullptr)
	{
		throw "Invalid name!";
	}
	this->name = new char[strlen(name) + 1];
	strcpy(this->name, name);
}
```

### Изключения в деструкторите

Изключенията не трябва да напускат деструкторите. Ако възникне изключение в деструктора, то трябва да се прихване и да се обработи, ако е възможно да бъдат освободени всички заделени външни ресурси. Проблемът при възникване на изключение в деструктор е, че при превъртането на стека при обработка на изключение се извикват деструкторите на всички локални обекти. Ако от такъв деструктор възникне изключение, то компилаторът не е в състояние да обработи новото изключение успоредно с текущото и възниква конфликт. В крайна сметка, програмата ще бъде прекъсната веднага без възможност за обработка.

---

## Нива на сигурност при изключенията

Съществуват няколко различни нива на сигурност при възникване на изключение. 

### No-throw guarantee
Операциите винаги са успешни. Дори и да възникне някакво изключение, то се обработва вътрешно и външният свят не разбира за него. Пример за такава функция е преместващият конструктор.

### Силна сигурност (strong exception safety guarantee)
Операциите могат да пропаднат, но ако дадена операция пропадне, то няма да има странични ефекти от нея и обектът остава в оригиналното си състояние. Понякога се постига по-трудно.

### Слаба сигурност (weak/basic exception safety guarantee)
Частично изпълнение на операциите, което може да доведе до странични ефекти. Обектът е във валидно състояние, което може да е различно от първоначалното. Няма изтичане на ресурси.

### No exception safety
Няма никаква гаранция за състоянието на обектите и операциите.

---

Да допуснем, че възниква изключение при заделянето на памет за name:  

**Пример за basic exception safety guarantee:**
```c++
Item& Item::operator=(const Item& other)
{
	if (this != &other) 
	{
		this->setID(other.id);
		this->setPrice(other.price);
		this->setName(other.name);
	}
	return *this;
}
```

**Пример за strong exception safety guarantee:**
```c++
Item& Item::operator=(const Item& other)
{
	if (this != &other) 
	{
		this->setName(other.name);
		this->setID(other.id);
		this->setPrice(other.price);
	}
	return *this;
}
```
