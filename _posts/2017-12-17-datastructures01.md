---
layout: post
title: Introduction to data structures (Implementing Lists)
published: false
---

This word can sound a bit theoretical and complex at first, but, what are data structures for a programmer?

In this serie of posts we are going to implement various typicall data structures and explaint some aspects about them.

# What are data structures for a programmer?
A data structure is a **class** used for organize data and provide various operations upon their data. 

# About Arrays?
The most famous data structure of all is the Array, surely all we have used it ever.

The arrays are a contiguous space in memory that should be the same **type**, also is good to mention that, the Array's writing, read operations are done in constant time, this is: O(1).

The arrays are a important data structure for other data structures as for example the **List**

Keep in mind that array of value types store its item in its value form, then no boxing operation is needed for read its items.

# The Lists
The List is a basic data structure which wrap up an basic **array** and empower it with the hability of auto-increment its size.

While an Array should get a constant size specified in its constructor, a List does not require this.

```c#
// You only are able to put 5 favourite numbers in this array.
int[] favouriteNumbers = new int[5];

// You do not know the total friend you will met in your life, so you can not specify the length of the array at programing time.
List<string> allTheFriendsOfMyLife = new List<string>();
```

Note that list keep its write/access time constant O(1), then they are not less fastest than arrays, however, the may get a bit more of space in RAM.

## C# Jargon
### Type-Safe Collection
Is a collection for example a **list** which ensures to the programmer client that only can have values of only one Type, this is not the case with the **System.Collection.ArrayList** class.

### About the ArrayList class
The ArrayList is a list which is not Type-Safe, since it is basically an array of Objects, since in C# all the types hinherit from the **Object** class this list is able to store any type.

But all is not perfect, **ArrayList** need make a Boxing operation for store any item, then ValueTypes as for example integers need to be boxed first for save it int the **ArrayList**, and also, Value Types need to be unboxed in every read operation of the **ArrayList**

```c#
ArrayList foo = new ArrayList();
foo.Add(5); // Need perform a boxing operation over 5

```


# Implementing the List
In this section, we are going to implement a basic type-safe list.

The code is commented as much as possible for failitate the code readability.
```c#
/* Although Lists have a bit of boilerplate for 
 * resize them its read, write operations are done in O(1) time.
 * 
 */
public class MyList<T>
{
    public T[] arr;

    public int incrementFactor = 2;

    /// <summary>
    /// Gets the quantity of items added to the list
    /// </summary>
    public int Count
    {
        get
        {
            return count;
        }
    }

    private int count;


    public MyList()
    {
        // Starts array with 4 elements by default.
        arr = new T[4];
    }

    // Overloads the constructor for make configurable the list's array' length
    public MyList(int size)
    {
        arr = new T[size];
    }

    /// <summary>
    /// Adds a new item to arr[Count]
    /// </summary>
    /// <param name="item">Item to add to the list</param>
    public void Add(T item)
    {
        // Checks if number of Items added is less than arr length
        if (count < arr.Length)
        {
            arr[count] = item;
        }
        else // if code reaches this, then count == to length, so the original array is full making needed increase its size.
        {
            // Double the arr capacity by default
            T[] tempArr = new T[arr.Length * incrementFactor];
            Array.Copy(arr, tempArr, arr.Length);
            // Change the reference of the variable arr to the same as the new array
            arr = tempArr;

            arr[count] = item;
        }
        count++;
    }

    /// <summary>
    /// Sets/Gets the item at specified index 
    /// </summary>
    /// <param name="index">Item index to select</param>
    /// <returns>The item at specified index</returns>
    /// 
    /* this notation is known as indexers, and allow us to make 
     * possible to the client code access the array using [] notation
     * NOTE: An indexor can have more that one formal parameter, as in the two dimensional arrays.s
     * 
     */
    public T this[int index] 
    {
        get
        {
            return arr[index];
        }

        set
        {
            // Checks user is accessing an valid Item
            /* count-1 because when count is equal to 0
             * index > count will be true because is true that index == count
             * 
             */

            if (index < 0 || index > (count - 1))
                throw new IndexOutOfRangeException();

            arr[index] = value;
        }
    }

    /// <summary>
    /// Removes first item from the list equal to the argument
    /// if operation was succesfull, then return true
    /// </summary>
    /// <remarks>Note that this function make a linear search O(n) for find the item in the list's array</remarks>
    /// <param name="item">The item to delete from the list</param>
    /// <returns></returns>
    public bool Remove(T item)
    {
        for (int i = 0; i < arr.Length; i++)
        {
            if (arr[i].Equals(item))
            {
                // TODO: Implement RemoveAt function
                RemoveAt(i);

                // Returns true if the item was found and succesfully deleted from the list's array
                return true;
            }
        }

        // Returns false if item element is not found in the list's array
        return false;                
    }

    /// <summary>
    /// Remove item located at specified index
    /// </summary>
    /// <param name="index">Index of item to be removed</param>
    public void RemoveAt(int index)
    {
        // Checks index out of ranges
        if (index < 0 || index > (count-1))
        {
            throw new IndexOutOfRangeException();
        }

        /* Array.Copy() is similar to c++ memmove() function,
         * then its complexity is O(n)
         * see: https://goo.gl/XUGPUj
         * 
         * Arra.Copy() is O(n) where n is the 'length' argument
         * 
         */

        /*
         * If list's arr have 4 elements and it is deleted the
         * element in index 2, then it is required calculated the 
         * number of elements from index 2 (exlusive) to the last index (incluse)
         * for calculate array's length
         * */

        int remaindLength = arr.Length - (index + 1);
        // T[] remaindElements = new T[remaindLength];

        /* Shift the elements located after specified index 
         * one index to left for replace the element located at arr[index]
         * */
        Array.Copy(arr, index + 1, arr, index, remaindLength);

        // Decrease the count variable not the length of the list's array
        count--;

    }
}
```

# References
- [An Extensive Examination of Data Structures Using C# 2.0](https://msdn.microsoft.com/en-us/library/ms379570%28vs.80%29.aspx?f=255&MSPPError=-2147217396)
- [How do ValueTypes derive from Object (ReferenceType) and still be ValueTypes?
](https://stackoverflow.com/questions/1682231/how-do-valuetypes-derive-from-object-referencetype-and-still-be-valuetypes)
