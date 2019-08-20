LRU cache :
  
#include <unordered_map>
#include <list>

using namespace std;

//https://leetcode.com/problems/lru-cache/
/*
146. LRU Cache
Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: get and put.
https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU
get(key) - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
put(key, value) - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.
Follow up:
Could you do both operations in O(1) time complexity?
Example:
LRUCache cache = new LRUCache(2); //capacity
cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // returns 1
cache.put(3, 3);    // evicts key 2
cache.get(2);       // returns -1 (not found)
cache.put(4, 4);    // evicts key 1
cache.get(1);       // returns -1 (not found)
cache.get(3);       // returns 3
cache.get(4);       // returns 4
Observations:
LRU cache based upon the idea that the least recently used cache is the first to be removed if have to. In order to
achieve O(1) on both get and put operation, a special designed data structure is needed to do:
1. Put new key in the cache in O(1);
2. Track the age of the cache in O(1);
3. Update the age ofthe cache if visited or updated in O(1);
4. Remove the oldest cache in O(1);
A hashtable to store key/value appears to be an option, age can be attached with value as a pair. However, updating age
is a concern. In this implmentation, every time a cache is used/updated, the ages of all other caches have to be updated
accordingly. That is an O(N) operation. Therefore, another data structure has to be introduced:
Deque, which allows adding/removing elements from both end and depending on the implementation, potentially removing an element
only takes O(1) if it's backed by a linked list. C++ STL list is such an implementation.
1. If a new element is added, push it to the front of the queue;
2. If an existing element is updated, removing it from the queue first and then push it to the front of the queue;
3. If have to remove an element (LRU), always pop from the end of the queue;
One can tell all three operations are O(1).
On the other hand, in hash table, key is the cache key, value is a pair of cache value and the node reference of the
element on the deque. In C++'s case, it's an iterator of list<int>. This is because the definition of std::list::erase
iterator erase (iterator position);
Which takes an iterator. This iterator is needed for updating cache. In this scenario, the updated cache (not LRU) can
be identified on the queue and removed in O(1) if hash table has the iterator stored as part of the value. Other than that,
1. Hash table adds cache key/value and the iterator after the cache key is pushed on to the queue, O(1)
2. Hash table removes chace key/value for LRU, O(1)
Java has a LinkdHashMap supports building LRU cache. Take a look on its removeEldestEntry protected method.
*/
class LRUCache {
private:
	int capacity;
	unordered_map<int, std::pair<int, list<int>::iterator>> cache;
	list<int> queue;
	void updateCacheAge(int key) {
		queue.erase(cache[key].second);
		queue.push_front(key);
		cache[key].second = queue.begin();
	}
public:
	LRUCache(int capacity) : capacity(capacity) {}

	int get(int key) {
		int ans = -1;
		if (1 == cache.count(key)) {
			ans = cache[key].first;
			this->updateCacheAge(key);
		}
		return ans;
	}

	void put(int key, int value) {
		if (1 == cache.count(key)) {
			cache[key].first = value;
			this->updateCacheAge(key);
		}
		else {
			if (this->capacity == queue.size()) {
				cache.erase(queue.back());
				queue.pop_back();
			}
			queue.push_front(key);
			cache.emplace(key, std::make_pair(value, queue.begin()));
		}
	}
};

/**
* Your LRUCache object will be instantiated and called as such:
* LRUCache obj = new LRUCache(capacity);
* int param_1 = obj.get(key);
* obj.put(key,value);
*/
/*
Test cases:
["LRUCache","put","get"]
[[1],[2,1],[2]]
["LRUCache","put","put","get","put","get","put","get","get","get"]
[[2],[1,1],[2,2],[1],[3,3],[2],[4,4],[1],[3],[4]]
["LRUCache","put","put","put","put","get","get"]
[[2],[2,1],[1,1],[2,3],[4,1],[1],[2]]
["LRUCache","get","put","get","put","put","get","get"]
[[2],[2],[2,6],[1],[1,5],[1,2],[1],[2]]
Outputs:
[null,null,1]
[null,null,null,1,null,-1,null,-1,3,4]
[null,null,null,null,null,-1,3]
[null,-1,null,-1,null,null,2,6]
*/

Request code:
#include <stddef.h>
#include <iostream>
#ifndef DEQUE_H

#define block_size(type) sizeof(type)*10000000
#define alloc(type) (type*)malloc(block_size(type))


#define Deque_DEFINE(type)\
  struct Deque_##type;\
  struct Deque_##type##_Iterator;\
  /*MAX_##type         : The maximum elements the deque can have before it reallocates
    CUR_BLOCK_COUNT    : The current block count which increases when reallocation occurs
    shift_count_##type : The number by which the elements which needs shifted will shift by in the new reallocated array
    load_factor_##type : The decision making value by which once the load factor % is reached, reallocation has to happen*/ \
  size_t MAX_##type = 10000000;\
  size_t CUR_BLOCK_COUNT_##type = 1;\
  const size_t shift_count_##type = 10000000;\
  const int load_factor_##type = 100;\
  \
  /*creating Deque_##type##_Iterator struct*/\
  typedef struct Deque_##type##_Iterator\
  {\
        type *value;\
  	int current_index;\
  	int frnt;\
  	int bck;\
  	type* data;\
    bool (*type##_comparator)(const type& o1, const type& o2);\
    \
    /*increment function which initially checks if the iterator is not currently pointing to the back, if not then the iterator increments.
      The index is calculated and the iter->value stores the value at that index.
      If the iterator is at the back, then we set the iterator index to -1 and the iter->value to NULL */	\
    void inc(Deque_##type##_Iterator* iter){\
    	if(iter->current_index != iter->bck){\
    		iter->current_index++;\
    		iter->current_index %= MAX_##type;\
    		iter->value = &iter->data[iter->current_index];\
    	}\
    	else{\
    		iter->current_index = -1;\
		iter->value = NULL;\
    	}\
    }\
    \
    /*decrement function which initially checks if the iterator is not currently pointing to the front, if not then the iterator decrements.
      The index is calculated and the iter->value stores the value at that index.
      If the iterator is at the front, then we set the iterator index to -1 and the iter->value to NULL.*/\
    void dec(Deque_##type##_Iterator* iter){\
    	if(iter->current_index != iter->frnt){\
    		if(iter->current_index == -1){\
    			iter->current_index = iter->bck;\
    			iter->value = &iter->data[iter->bck];\
    			return;\
    		}\
    		iter->current_index = iter->bck;\
    		iter->current_index = (iter->current_index-1+MAX_##type)%MAX_##type;\
    		iter->value = &iter->data[iter->current_index];\
    	}\
    	else{\
		iter->current_index = -1;\
		iter->value = NULL;\
    		}\
    	}\
    \
   /* deref function dereferences the iterator and returns the value pointed to by the iterator*/\
    type& deref(Deque_##type##_Iterator* iter){\
    	return *iter->value;\
    }\
  }iter_##type;\
  \
  iter_##type *b_it_##type, *e_it_##type;\
  \
  /*Creating Deque_##type structure*/\
  struct Deque_##type{\
    type* data;\
    int  bck,frnt;\
    int number_of_elements;\
    const char type_name[sizeof("Deque_"#type)] = "Deque_"#type;\
    bool (*type##_comparator)(const type& o1,const type& o2);\
    \
   /*clear function frees the data and the sets the front,back and number of elements of the deque to -1,-1 and 0 respectively. 
     Also, it sets the data back to its original size which can be used again.*/\
    void clear(Deque_##type *q)\
    {\
        free(q->data);\
	q->bck = -1;\
	q->frnt=-1;\
	q->number_of_elements = 0;\
        q->data = alloc(type);\
    }\
    /*Destructor which frees the data.*/\
    void dtor(Deque_##type *q)\
    {\
	free(q->data);\
    }\
    \
   /*empty function checks if the Deque is empty by making a check at the back index being -1. If yes , return true otherwise return false*/\
    bool empty(Deque_##type *q)\
    {\
    	if(q->bck == -1)\
    		return true;\
    	else\
    		return false;\
    }\
    \
   /*push_back function pushes the element of type ##type to the back of the Deque.
     So, first we check the boundary condition which being the Deque is empty, if yes, we set the front & back to 0 as 0th index is first element index
     and we store the element in the data at index 0
     Otherwise, if the Deque is not empty, then we check if we have reached the load factor, if yes then we need to reallocate and perform shifting if required nd then add the element in data.
     Otherwise, if the load factor is reached but the back is greater than front, then we will need to reallocate but no shifting operations are required.
     Finally, if the load factor is not reached, then we just add the element in the data and change our back to that element. We also increment the number of elements.*/\
    void push_back(Deque_##type *q,type x)\
    {\
      if(empty(q))\
    	{\
    		q->bck=0;\
    		q->frnt = 0;\
    		q->data[0] = x;\
        q->number_of_elements++;\
        return;\
    	}\
    	else\
    	{\
        if(q->number_of_elements == (load_factor_##type/100.0 * MAX_##type)){\
          q->data = (type*)realloc(q->data, (block_size(type))*(++CUR_BLOCK_COUNT_##type));\
          /*
          if bck == (load_factor_##type% * MAX_##type) - 1
            no need to shift
          else
            if bck > frnt
              no need to shift
            else
              shift everything from bck+1 to the end
            endif
          endif
          */\
          if(!(q->bck == ((load_factor_##type/100.0) * MAX_##type)))\
          {\
            if(q->bck < q->frnt){\
              int i;\
              for(i = MAX_##type-1; i >= q->frnt; i--)\
              {\
                q->data[i+shift_count_##type] = q->data[i];\
              }\
              /*re-assigning new frnt*/\
              q->frnt += shift_count_##type;\
            }\
          }\
          MAX_##type = ((block_size(type))*(CUR_BLOCK_COUNT_##type))/sizeof(type);\
      	}\
        q->bck = (q->bck+1)%(MAX_##type);\
        q->data[q->bck] = x;\
    	}\
      q->number_of_elements++;\
    }\
    /*push_front is the function which puts the element of type ##type to the front of the Deque.
      First we check if the boundary condition being the Deque is empty is reached. Then we add the element to index 0 of data after setting the front and back to 0.
      If the Deque is not empty, the we check if the load factor is reached , if yes then we check if the back is less than front, if yes then we need to reallocate 
      and do shifting and then add the element.
      Otherwise, if back is greater than front then we perform reallocation but no shifting is performed and finally the data is added to the front.
      Otherwise, if the load factor is not reached, then we just change the front and then add the element to the front of the Deque. We also increment the number of elements.*/\
    void push_front(Deque_##type *q,type x)\
    {\
    	if(empty(q))\
    	{\
    		q->bck = 0;\
    		q->frnt = 0;\
    		q->data[0] = x;\
		q->number_of_elements++;\
        return;\
    	}\
    	else\
      {\
        if(q->number_of_elements == (load_factor_##type/100.0 * MAX_##type))\
        {\
            q->data = (type*)realloc(q->data, (block_size(type))*(++CUR_BLOCK_COUNT_##type));\
            /*
            if bck == (load_factor_##type% * MAX_##type) - 1
              no need to shift
            else
              if bck > frnt
                no need to shift
              else
                shift everything from bck+1 to the end
              endif
            endif
            */\
            if(!(q->bck == ((load_factor_##type/100.0) * MAX_##type)))\
            {\
              if(q->bck < q->frnt){\
                int i;\
                for(i = MAX_##type-1; i >= q->frnt; i--)\
                {\
                  q->data[i+shift_count_##type] = q->data[i];\
                }\
              }\
            }\
            MAX_##type = (block_size(type)*(CUR_BLOCK_COUNT_##type))/sizeof(type);\
            /*re-assigning new frnt*/\
            q->frnt += shift_count_##type;\
      	}\
        q->frnt = (q->frnt-1+(MAX_##type))%MAX_##type;\
        q->data[q->frnt]=x;\
      }\
      q->number_of_elements++;\
    }\
    \
    /*pop_front basically pops an element from the front of the Deque.
      First we make a check at the boundary condition being empty, if yes then we just return.
      Otherwise, if front is equal to back , then we set front , back and number of elements to -1,-1 and 0 respectively.
      Otherwise,we change our front pointing now to last but one to front and we reduce the count of the number of elements. */\
    void pop_front(Deque_##type *q)\
    {\
      if(q->empty(q)){\
        return;\
      }\
    	if(q->bck == q->frnt)\
    	{\
			q->bck =-1;\
      			q->frnt = -1;\
      			q->number_of_elements = 0;\
      			q->data = alloc(type);\
    	}\
    	else\
    		q->frnt = (q->frnt+1)%(MAX_##type);\
    	q->number_of_elements--;\
    }\
    \
   /*pop_back basically pops an element from the back of the Deque.
     First we make an empty check, if it passes then we just return.
     Otherwise, we check if the front and back are the same, if yes then we set front, back and number of elements to -1,-1 and 0 respectively.
     Otherwise, if the back is not equal to front, we make our back point to last but one from current back and reduce then number of elements count. */\
    void pop_back(Deque_##type *q)\
    {\
      if(q->empty(q)){\
        return;\
      }\
    	if(q->bck == q->frnt)\
    	{\
			q->bck =-1;\
      			q->frnt = -1;\
      			q->number_of_elements = 0;\
      			q->data = alloc(type);\
    	}\
    	else\
    		q->bck = (q->bck-1+(MAX_##type))%(MAX_##type);\
    	q->number_of_elements--;\
    }\
    \
    /*front function returns the element at the front of the Deque.*/\
    type& front(Deque_##type *q)\
    {\
        return q->data[q->frnt];\
    }\
    \
    /*back function returns the element at the back of the Deque.*/\
    type& back(Deque_##type *q)\
    {\
      return q->data[q->bck];\
    }\
    \
    /*begin function basically sets the iterator to the front of the Deque.*/\
    iter_##type& begin(Deque_##type *q)\
    {\
    	 type &f=front(q);\
    	 b_it_##type = (Deque_##type##_Iterator*)malloc(sizeof(Deque_##type##_Iterator));\
    	 b_it_##type->value = &f;\
    	 b_it_##type->frnt = q->frnt;\
    	 b_it_##type->bck = q->bck;\
    	 b_it_##type->current_index = q->frnt;\
    	 b_it_##type->data = q->data;\
         b_it_##type->type##_comparator = q->type##_comparator;\
    	 return *b_it_##type;\
    }\
    \
   /*end function sets the current index to -1 which is one past the last element in the Deque.*/\
    iter_##type& end(Deque_##type *q)\
    {\
	e_it_##type = (Deque_##type##_Iterator*)malloc(sizeof(Deque_##type##_Iterator));\
    	e_it_##type->frnt = q->frnt;\
    	e_it_##type->bck = q->bck;\
    	e_it_##type->current_index = -1;\
    	e_it_##type->data = q->data;\
        e_it_##type->type##_comparator = q->type##_comparator;\
    	return *e_it_##type;\
    }\
    \
   /*size function returns the number of elements currently in the Deque.*/\
    int size(Deque_##type *q)\
    {\
    	return q->number_of_elements;\
    }\
    /*at function returns the position at a particular position after calculating index.*/\
    type& at(Deque_##type* q,int pos)\
    {\
    	int index = (q->frnt+(pos))%(MAX_##type);\
    	return q->data[index];\
    }\
  };\
  \
  /*Constructor which initializes the Deque.Also set the function pointer to the comparator function.*/\
  void Deque_##type##_ctor(Deque_##type *q,bool (*fptr)(const type& o1, const type& o2))\
  {\
    q->bck =-1;\
    q->frnt = -1;\
    q->number_of_elements = 0;\
    q->data = alloc(type);\
    q->type##_comparator = fptr;\
  }\
  \
 /*This function checks if the two iterators are equal.*/\
  bool Deque_##type##_Iterator_equal(Deque_##type##_Iterator& iter1, Deque_##type##_Iterator& iter2){\
     if(iter1.current_index != iter2.current_index){\
      return false;\
    }\
	return true;\
  }\
\
  /*This function checks whether the two Deque's are equal.So basically if they have the same number of elements , only then we go ahead
    and loop through both the Deque's and check if the elements are the after making use the comparators provided by the professor in his test file.
    If the number of elements are different , then we just return false.*/\
  bool Deque_##type##_equal(Deque_##type& q1, Deque_##type& q2){\
  	if(q1.number_of_elements != q2.number_of_elements){\
  		return false;\
  	}\
  	else{\
		int i;\
		for(i=0;i<q1.number_of_elements;i++)\
		{\
			if(q1.type##_comparator(q1.at(&q1,i), q2.at(&q2,i)) || q1.type##_comparator(q2.at(&q2,i), q1.at(&q1,i)))\
			{\
				return false;\
			}\
		}\
  		return true;\
  	}\
  }\

#endif
