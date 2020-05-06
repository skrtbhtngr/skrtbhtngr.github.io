Many users who are beginning a new open-source project, or are making their existing projects open-source face the difficulty in choosing a license for their project. The legal jargons inside those licenses might go over your head.

In this post, I'll try to explain the most frequently used licenses in very simple terms. These terms might NOT be 100% legally accurate as they are meant to just provide you with a gist of the various licenses. I will focus on the basic rules and leave out any complexities. For a brief comparison, visit [here](https://en.wikipedia.org/wiki/Comparison_of_free_and_open-source_software_licenses#General_comparison).

There are many open-source licenses out there which you can add to your project, and these vary mainly in the type of *freedom* you (the **Licensor**) are giving to the person who may *use* your code (the **User**). Did you see that? I just defined two terms used very often in legal notices and licenses. But, why stop at this? Let's bring some more terms into play. Your project (for which you want a license) will be called as **Work**, because, it is your work! Also, when a User uses your code and makes some changes, we will call that modified code as **Derived Work**.

Also, the term **Permissive license** means that there are very few requirements from how your Work might be redistributed by another User, and the term **Copyleft** is used to refer to a practice wherein the User is expected to preserve the same license term in his/her Derived Work. The extent to which the Copyleft is imposed by the license is referred to as the strength (**Strong** or **Weak**) of the Copyleft.

[This](https://choosealicense.com) is a nice website that helps you choose a license.


___
### Apache License, Version 2.0 [[link](https://www.apache.org/licenses/LICENSE-2.0)]

This is a *permissive* (or *non-protective*) license, also known as a *weak copyleft* license. Basically, the Users can do *anything* they want with your code, that is

* use it in their project,
* sell it and maybe earn some money! , 
* modify it and sell it,
* modify it and release it (maybe with some other license),
* modify it and use it in a proprietary software!,
* *add* their own copyright to the modifications they made,
* play with it,
* make wall posters of it,
* any combination of the above;

literally, anything. But, here's the catch: there are some things that they *have* to do:

* they must include a copy of this license in their Derived Work.
* if they modify a file from your Work, they must explicitly state this in that file.
* they must include the copyright and trademark info in your Work; that is, give you some credits.
* cannot use your trademark or product name, except when mentioning the source of their Derived Work.
* cannot hold you liable if your code breaks.

Version 2.0 allows you to include the copy once in your project instead of including it in every file.

Some notable software and libraries using this license:

* Android 
* Docker
* IntelliJ IDEA and PyCharm
* .NET CCompiler Platform
* TensorFlow


___
### MIT license [[link](https://opensource.org/licenses/MIT)]

This is one of the most easy-to-understand licenses available, and its license statement is very concise. This also a *permissive* (or *non-protective*) license also known as a *weak copyleft* license. The Users can pretty much do *anything* with your code (just like in Apache license), and only a few things are expected of them:

* they must include a copy of this license in their Derived Work.
* they must include the copyright info in your Work; that is, give you some credits.
* cannot hold you liable if your code breaks.


Some notable software and libraries using this license:

* AngularJS
* Bitcoin Core
* GitLab
* jQuery
* Ncurses

___
### GNU General Public License, version 2 [[link](https://www.gnu.org/licenses/gpl-2.0.html)]

This a *protective* or *strong copyleft* license, which means that any Derived Work must be distributed under the same license. One of the key things to note here is that any User of your Work is bound to make his/her (relevant portion of) Derived Work open-source! Apart from that, Users can pretty much do anything with your code (as mentioned in the Apache license section above).

The following is expected from the Users:

* they must give the Users of their Derived Work all the rights they have (including the source code!).
* if they are using the Derived Work as a part of a bigger project, which is dependent on the Derived Work, the whole project must exercise this license!
* if they modify a file from your Work, they must explicitly state this in that file (along with the date of change).
* cannot hold you liable if your code breaks.


Some notable software and libraries using this license:

* Adblock Plus
* Code::Blocks
* Gedit
* Git
* Weka


___
##### NOTE: If you find any mistake or legally incorrect statement, please feel free to contact me at skrtbhtngr@gmail.com.
