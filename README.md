# SOLID-Principle
# Introduction

The SOLID principle was introduced by Robert C. Martin, also known as Uncle Bob and it is a coding standard in programming.SOLID principles are some of the oldest rules in the software world. They enable us to write maintainable, readable, reusable code. In this text, I am trying to accomplish a somewhat real-life example, obeying the SOLID principles.This principle is an acronym of the five principles which is given below…
* Single Responsibility Principle (SRP)
* Open/Closed Principle
* Liskov’s Substitution Principle (LSP)
* Interface Segregation Principle (ISP)
* Dependency Inversion Principle (DIP)


# Single Responsibility Principle (SRP)

This principle states that “a class should have only one reason to change” which means every class should have a single responsibility or single job or single purpose.Consider the following example:

``` java 
    public class PasswordHasher
{
    public String hashAndSavePassword(String password)
    {
        hashPassword();
        savePassword();
    }

    public void hashPassword()
    {
        //hashing implementation
    }
    public void savePassword()
    {
        //save to the db
    }
}
```
This class is implemented for hashing passwords, as the name implies. It should not be its responsibility to save them to the database. Each class should have a single responsibility to fulfill.

There should not be “god classes” that have a broad variety of functionality that has too much to accomplish. Instead, we should write our classes as modular as possible. Implement the saving operation in another class.

# Open-Closed Principle

This principle states that “software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification” which means you should be able to extend a class behavior, without modifying it.

Let’s continue to our password-hasher example. Suppose we want our class to be able to hash with a variety of algorithm options.

```java 
public String hashPassword(String password, HashingType hashingType)
{
    if(HashingType.BASE64.equals(hashingType))
    {
        hashedPassword="hashed with Base64";
    }
    else if(HashingType.MD5.equals(hashingType))
    {
        hashedPassword="hashed with MD5";
    }

    return hashedPassword;
}
```
# Liskov’s Substitution Principle

The principle was introduced by Barbara Liskov in 1987 and according to this principle “Derived or child classes must be substitutable for their base or parent classes“. This principle ensures that any class that is the child of a parent class should be usable in place of its parent without any unexpected behavior.

To demonstrate our example, let’s create the Model (Data) Classes to use our hashing algorihms.
```java 
public abstract class Hashed
{
    PasswordHasher passwordHasher;
    String hash;
    
    public void generateHash(String password)
    {
        hash = passwordHasher.hashPassword(password);
    }
}
```
```java 
public class Base64 extends Hashed
{
    public Base64()
    {
        this.passwordHasher = new Base64Hasher();
    }
}
```
To fulfill Liskov’s Rule, each other extension of Hashed should use a valid implementation of hashing function and return a hash.

For example, if we extend the Hashed class with a class called “NoHash” that uses an implementation that returns exactly the same password without any encoding will break the rule, since a subclass of Hashed is expected to have a hashed value of the password.


# Interface Segregation Principle

This principle is the first principle that applies to Interfaces instead of classes in SOLID and it is similar to the single responsibility principle. It states that “do not force any client to implement an interface which is irrelevant to them“. Here your main goal is to focus on avoiding fat interface and give preference to many small client-specific interfaces. You should prefer many client interfaces rather than one general interface and each interface should have a specific responsibility.

Consider we add decoding feature to the interface.
```java 
public interface PasswordHasher
{
    String hashPassword(String password);
    String decodePasswordFromHash(String hash);
}
```
This would break this law since one of our algorithms, the SHA256 is not decryptable practically, (it’s a one-way function). Instead, we can add another interface to the applicable classes to implement their decoding algorithm.
```java
public interface Decryptable
{
    String decodePasswordFromHash(String hash);
}
```
```java
public class Base64Hasher implements PasswordHasher, Decryptable
{
    @Override
    public String hashPassword(String password)
    {
        return "hashed with base64";
    }

    @Override
    public String decodePasswordFromHash(String hash)
    {
        return "decoded from base64";
    }
}
```
# Dependency Inversion Principle

In this principle, they are two key points
*High-level modules/classes should not depend on low-level modules/classes. Both should depend upon abstractions.
*Abstractions should not depend upon details. Details should depend upon abstractions.
You can consider the real-life example of a TV remote battery. Your remote needs a battery but it’s not dependent on the battery brand. You can use any XYZ brand that you want and it will work. So we can say that the TV remote is loosely coupled with the brand name. Dependency Inversion makes your code more reusable.
We have a password service like the following:
```java 
public class PasswordService
{
    private Base64Hasher hasher = new Base64Hasher();
    void hashPassword(String password)
    {
        hasher.hashPassword(password);
    }
}
```
We violated the principle since we tightly coupled the Base64Hasher and PasswordService.

Let’s decouple them and let the client inject the hasher service needed with the constructor.
```java
public class PasswordService
{
    private PasswordHasher passwordHasher;
    
    public PasswordService(PasswordHasher passwordHasher)
    {
        this.passwordHasher = passwordHasher;
    }
    
    void hashPassword(String password)
    {
        this.passwordHasher.hashPassword(password);
    }
}
```
Much better. We can easily change the hashing algorithm. Our service does not care about the algorithm, it's up to the client to choose it. We don’t depend on the concrete implementation, but the abstraction.

# References
https://www.baeldung.com/solid-principles

https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design

https://www.geeksforgeeks.org/solid-principle-in-programming-understand-with-real-life-examples/
