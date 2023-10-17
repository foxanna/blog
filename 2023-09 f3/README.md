# Mobile apps distribution with Firebase

*An in-person talk by Anna and Oleksandr Leushchenko at the [Firebase Flutter Festival](https://f3.events/) conference on September 27, 2023.*

![](images/cover_image.png)

Having a well-defined release strategy is important for the success of any commercial mobile product. A good strategy should include some form of distributing release-candidate builds to the testers and internal users before uploading them to app stores for general availability. This allows catching problems earlier which dramatically improves the quality of the final product. 

Firebase Distribution is a great way to solve this task: it's easy to use, secure, and reliable. First, we’ll look into using Firebase Distribution via web interface to create testing groups and distribution rings, and to share builds internally. Automating build distribution is not an easy process, but it’s a time and effort-saver in the long run. That’s why we'll demonstrate how to use the Firebase CLI tool and Fastlane to automate this process on CI.
