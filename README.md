
# Java-FSE

## Resources:
### OOP 
1. https://medium.com/javarevisited/preparing-for-a-java-interview-here-are-the-top-19-oops-questions-that-you-need-to-know-6cc6dde3d4c3



### SpringBoot
1. N+1 Query Problem: https://medium.com/@kiarash.shamaii/what-is-n-1-query-generate-problem-in-spring-data-jpa-and-how-to-solve-it-2f3b9f1a7a0b 

2. Auditing Spring Boot: https://medium.com/thefreshwrites/jpa-auditing-spring-boot-spring-security-575c77867570

||`Page<T>`|`Slice<T>`|
|---|---|---|
|Total count query|✅ Yes (extra COUNT query)|❌ No|
|`getTotalPages()`|✅|❌|
|`getTotalElements()`|✅|❌|
|`hasNext()`|✅|✅|
|Performance|Slightly slower|Faster (no COUNT)|
|Use case|Traditional pagination with page numbers|Infinite scroll / "load more"|