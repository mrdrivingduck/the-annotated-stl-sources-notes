# Chapter 4.5 - stack

Created by : Mr Dk.

2021 / 04 / 04 14:44 ð

Nanjing, Jiangsu, China

---

## 4.5.1 stack æ¦è¿°

stack æ¯ä¸ç§ FILO (First In Last Out) çæ°æ®ç»æï¼åªæä¸ä¸ªåºå£ãåè®¸ä»¥ä¸æä½ï¼

* æ°å¢åç´ 
* ç§»é¤åç´ 
* åå¾æé¡¶ç«¯åç´ 

æ­¤å¤ï¼æ²¡æå¶å®æ¹æ³å¯ä»¥å­å stack ä¸­çåç´ ï¼ä¹æ²¡æéåæä½ã

## 4.5.2 stack å®ä¹å®æ´åè¡¨

deque æ¯ååå¼å£çæ°æ®ç»æãå¦æä»¥ deque ä¸ºåºå±ç»æï¼å°é­ deque çå¤´é¨å¼å£ï¼å°±å½¢æäºä¸ä¸ª stackãSGI STL é»è®¤ä»¥ deque ä¸º stack çåºå±ç»æãstack çææ API é½ç±åºå±æ°æ®ç»æç API æ¥å®ç°ï¼åªä¸è¿å±è½äºæäºåºå±ç»æç API (æ¯å¦å¤´é¨å¼å£)ãå·æè¿ç§ *ä¿®æ¹æç©æ¥å£ï¼å½¢æå¦ä¸ç§é£è²* æ§è´¨çç»æè¢«ç§°ä¸º **adapter (ééå¨)**ãSTL ä¸­ç stack éå¸¸ä¸è¢«å½ç±»ä¸ºå®¹å¨ (container)ï¼èæ¯å®¹å¨ééå¨ (container adapter)ã

```c++
template <class _Tp, 
          class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(deque<_Tp>) > // deque
class stack;
```

```c++
template <class _Tp, class _Sequence>
class stack {

  // requirements:

  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_REQUIRES(_Sequence, _BackInsertionSequence);
  typedef typename _Sequence::value_type _Sequence_value_type;
  __STL_CLASS_REQUIRES_SAME_TYPE(_Tp, _Sequence_value_type);


#ifdef __STL_MEMBER_TEMPLATES
  template <class _Tp1, class _Seq1>
  friend bool operator== (const stack<_Tp1, _Seq1>&,
                          const stack<_Tp1, _Seq1>&);
  template <class _Tp1, class _Seq1>
  friend bool operator< (const stack<_Tp1, _Seq1>&,
                         const stack<_Tp1, _Seq1>&);
#else /* __STL_MEMBER_TEMPLATES */
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const stack&, const stack&);
  friend bool __STD_QUALIFIER
  operator< __STL_NULL_TMPL_ARGS (const stack&, const stack&);
#endif /* __STL_MEMBER_TEMPLATES */

public:
  typedef typename _Sequence::value_type      value_type;
  typedef typename _Sequence::size_type       size_type;
  typedef          _Sequence                  container_type;

  typedef typename _Sequence::reference       reference;
  typedef typename _Sequence::const_reference const_reference;
protected:
  _Sequence c; // åºå±å®¹å¨
public:
  stack() : c() {}
  explicit stack(const _Sequence& __s) : c(__s) {}

  // ä»¥ä¸ API å¨é¨è½¬èè°ç¨åºå±å®¹å¨ç API å®ç°ç¸åºåè½
  // åªæ´é²åºå±å®¹å¨çå°¾é¨æ¥å£
  bool empty() const { return c.empty(); }
  size_type size() const { return c.size(); }
  reference top() { return c.back(); }
  const_reference top() const { return c.back(); }
  void push(const value_type& __x) { c.push_back(__x); }
  void pop() { c.pop_back(); }
};
```

è¿ç®ç¬¦éè½½ä¹ç´æ¥è°ç¨åºå±å®¹å¨çè¿ç®ç¬¦ï¼

```c++
template <class _Tp, class _Seq>
bool operator==(const stack<_Tp,_Seq>& __x, const stack<_Tp,_Seq>& __y)
{
  return __x.c == __y.c;
}

template <class _Tp, class _Seq>
bool operator<(const stack<_Tp,_Seq>& __x, const stack<_Tp,_Seq>& __y)
{
  return __x.c < __y.c;
}
```

## 4.5.3 stack æ²¡æè¿­ä»£å¨

stack ä¸æä¾éååè½ï¼ä¹ä¸æä¾è¿­ä»£å¨ã

## 4.5.4 ä»¥ list ä½ä¸º stack çåºå±å®¹å¨

list ä¹æ¯ååå¼å£çç»æï¼å¹¶ä¸ stack ä½¿ç¨çåºå±å®¹å¨æä½ `empty()` / `size()` / `back()` / `push_back()` / `pop_back()` list ä¹é½æãå æ­¤ä¹å¯ä»¥ä½¿ç¨ list ä½ä¸º stack çåºå±ç»æï¼

```c++
stack<int, list<int> > list_stack;
```

---

