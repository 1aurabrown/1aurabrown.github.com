---
layout: post
title: "Ruby Iterators: Enumerable Methods, Part I"
date: 2013-02-26 00:00
comments: true
categories: ruby code enumerable
---

\#each and \#map
----------

As someone new to Ruby who has a fairly limited lexicon of method names, I often have no idea that Ruby already contains a method to accomplish exactly what I am trying to achieve. Sometimes when I'm writing code, after constructing a sequence of method calls or wrapping some logic in my own method, I then find that Ruby has a built in method to abstract a common pattern, producing the same result in just one or two lines of code.



A few methods that I've recently come to understand are the <code>map</code>, <code>inject</code> and <code>each\_with\_object</code> methods. Anything that can be accomplished with these methods could also be accomplished in several more lines of code using a varation on the <code>each</code> method, so they are by no means absolutely necessary, but they have much more precise uses than the more general <code>each</code> method. Building these three methods from scratch using <code>each</code> helped me to understad (1) how they worked and (2) how easy it is to assign the result of some code to a variable and (3) how "yield" works. Today I'm going to examine <code>each</code> and <code>map</code>.

\#each
------
<code>each</code> is our basic iterator. It pretty much does what the name implies. To print the elements of an array, we can write:

<pre>
array = [1, 2, 3, 4, 5]
array.each do |i|
  puts i
end
</pre>

the result:
<pre>
1
2
3
4
5
 => [1, 2, 3, 4, 5] 
</pre>

<code>each</code>, our iterator, takes each element from its reciever, the Enumerable <code>array</code>, and yields it to the block, the indented line between our <code>each</code> call and the <code>end</code> keyword. It does this five times, as there are 5 elements in the array. With each iteration, the element currently passed to the block will temporarily be assigned to <code>i</code>. <code>end</code> indicates the end of the code to be executed with each iteration and signals the start of the following iteration using the next element in the reciever.

Even the <code>each</code> method could be written in more basic terms:
<pre>
array = [1, 2, 3, 4, 5]
i=0
while i < array.length
  puts array[i]
  i = i+1
end
</pre>  

produces:
<pre>
1
2
3
4
5
 => nil 
</pre>

This, isn't a method, it's just some code. Notice how both print the elements of the array, but our first code sample, using <code>each</code> returned the original array (as indicated by the hashrocket <code>=></code>), while the second returned nil. <code>each</code> <b>always</b> returns the origal array on which it was called. In both the above cases, we were simply printing each element of the array, not trying to modify it in any way. But what if we wanted to transform the array in some way? For example:

<pre>
array = [1, 2, 3, 4, 5]
array.each do |i|
  i + 5
end
=> [1, 2, 3, 4, 5] 
</pre>

Once again, <code>each</code> returns the orginal array. In order to create an array with new values, each five more than the values of the original array, we would have to do the following:

<pre>
array = [1, 2, 3, 4, 5]
result_array = []
array.each do |i|
  result_array << i + 5
end
 => [1, 2, 3, 4, 5] 
result_array
 => [6, 7, 8, 9, 10]
</pre>

The value of the variable <code>array</code> has not changed, but we now have a new array, <code>result\_array</code> containing the modified values. Note that the variable <code>result\_array</code> must be declared and assigned the value of empty array before <code>array.each</code> is called. Then with each iteration, a new value <code>i + 5</code>is appended to <code>result_array</code>.

\#map
---------
Why should we have to declare an array outside the <code>each</code> loop and then individually add values to it? We can't just assign the return of the <code>each</code> block to a new variable because, as we know, <code>each</code> always returns the <b>original</b> array. Ruby's <code>#map</code> method is built to do exactly what we are asking <code>each</code> to do!

<pre>
array = [1, 2, 3, 4, 5]
array.map do |i|
  i+5
end
 => [6, 7, 8, 9, 10] 
</pre>

Unlike <code>each</code>, <code>map</code> returns an array whose elements are the result of whatever code was evaluated within the block, which is what we really want. Take note that <code>map</code> doesn't "overwrite" the original array with the return array, but we can easily assign this return array to a variable.

<pre>
array = [1, 2, 3, 4, 5]
result_array = array.map do |i|
  i+5
end
 => [6, 7, 8, 9, 10] 

array
 => [1, 2, 3, 4, 5] 

result_array
 => [6, 7, 8, 9, 10]
</pre>

I sometimes forget how easy it is to assign the return value of an entire block of code to a variable because it's not as intuitive as writing <code>n = 1</code>, but it's really no different than assigning an integer, array, string or any other class of object to a variable. If you did not want to retain the original array and wanted to assign it the value of the new array, you could easily do so:

<pre>
array = [1, 2, 3, 4, 5]
array = array.map do |i|
  i+5
end
 => [6, 7, 8, 9, 10] 

array
 => [6, 7, 8, 9, 10]
</pre>

The difference between <code>map</code> and <code>each</code> is fairly straight-forward. We already say how certain uses of <code>each</code> are really begging for <code>map</code>. This was pretty easy to recognize, but building my own method that mimicked the functionality of <code>map</code> was a bit harder. The method can only  use the <code>each</code> method within its definition to iterate over the reciever array. It also must be versatile--like <code>map</code> it must yield to a block that specifies exactly what should happen to each element in the reciever--I don't always want to add 5 to every element. It must then return the modified array.

<pre>
array = [1,2,3,4,5]

def my_map
  return_array = []
  self.each do |n|
    return_array << yield(n)
  end
  return_array
end

array.my_map do |i|
  i * 2
end
=> [2, 4, 6, 8, 10] 
</pre>

The thing I really struggled to understand was how exactly <code>yield</code> makes this work. Whe we call <code>array.my\_map</code>, Ruby immediately shifts to the first line of code within the definition of <code>my\_map</code>. Before even interpreting <code> do |i|</code>, Ruby inserts a bookmark and starts reading the first line of the definition of <code>my_app</code>. On this line, a new empty array is initialized. On the next, the method <code>each</code> is called on <code>self</code>. In the context of a method definition, self is equal to the method's reciever. In this case, the reciever is <code>array</code>. In this particular method call, this line of code means <code>array.each do |n|</code>.

Like always, <code>each</code> signals an iteration over its reciever where each element in that reciever will temporarily be assigned to a variable, in this case <code>n</code>, and then passed to the block that follows. In the first iteration, <code>n</code> is equal to 1.

Inside the block on the following line, we see that some value is appended to the array <code>return_array</code>, but Ruby does not yet knwo what that value will be. <code>yield(n)</code> puts another bookmark in our code, and tells Ruby to go back to where it left of earlier, just before <code>do |i|</code>, bringing the variable <code>n</code>with it. The variable <code>i</code> is temporarily assigned the value of <code>n</code>. If this is our firt iteration, <code>n</code> is equal to 1. <code>i</code>is equal to <code>n</code> within this block. The block is then evaluated. The last line of code evaluated within the block is its implied return value.

In the first iteration, starting from the point where Ruby interprets <code>yield(n)</code>, here's an expanded version of what's happening:

<pre>
n = 1     
i = n
i * 2
 => 2 
</pre>

After this, Ruby reads the keyword <code>end</code> and knows that it has reached the end of the block. Thus the return value of the block is equal to 2 because the last line of code in the block evaluates to 2. Think back to when Ruby placed its last bookmark. It was back inside a different block of code, within the <code>my_map</code> method. Ruby paused evaluation at this point because it interpreted instructions to yield. Now that the block has ended, Ruby returns to this point. <code>yield(n)</code>evaluates to the return value of the block to which it yielded. In this case, <code>yield(n)</code> where n = 1 evaluates to 2, as we saw. Ruby now knows what value to append to the reciever of the shovel method, <code>return\_array</code>.

Upon interpreting the <code>end</code>keyword on the following line, Ruby will perform the next iteration over <code>array</code>, again assigning <code>n</code> a value, passing that value to the block where <code>my\_map</code> was originally called, it will evaluate that block and shovel its return value into <code>return\_array</code> all over again and again for each array element. After the last iteration, Ruby will proceed to the line following the <code>end</code> key. Because we want the method to return the modified array, the last line of code in the method definition must be <code>return_array</code>. Whenever this function is called, it will return the updated array.

Abstracting behaviors that will be used again and again is always a good decision. Utilizing yield in this method allowed us to abstract everthing execept the exact way in which each element of the original array would be transformed. The distracting and confusing logic of iteration and return values is separated from the code where the method is called, where all we really want is the result of all that logic. Using <code>yield</code> makes the method highly versatile--the logic is standardized, but we can specify the way in which the elements of the original array should be transformed. So long as we want to iterate over an array and return a new modified array, this method is applicable. This is of course why the makers of Ruby included it!

There are other methods that can be called on Enumerable objects that behave in slightly different ways. They follow a similar model--iteration and return values are abstracted. These too can be built using <code>each</code>. Building these methods from scratch helps to understand the subtle differences between them and. For me, it is a useful way to practice using <code>yield</code> and to add another build in Ruby method to my vocabulary.