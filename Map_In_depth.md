# HashMap Tree Bins (JDK 8+) — Insertion vs Lookup (Corrected Deep Dive)
---

## 1. When Does Treeification Happen?

A bucket (index in the table array) is converted from a **linked list** to a **Red-Black Tree** when **both** conditions are met:

- Number of nodes in a bucket ≥ 8 (TREEIFY_THRESHOLD)
- Table capacity ≥ 64 (MIN_TREEIFY_CAPACITY)

If capacity < 64, HashMap **resizes instead of treeifying**.

---

## 2. Insertion Before Treeification (Linked List)

```
bucket[i] -> (k1,v1) -> (k2,v2) -> (k3,v3) -> ...
```

---

## 3. Treeification (Linked List → Red-Black Tree)

Only one bucket is converted.  
This is **not TreeMap**, but an internal Red-Black Tree (`TreeNode`).

---

## 4. Tree Construction (Insertion Phase)

Ordering rules:
1. hash          --> this spreaded one not hashCode() one 
2. Comparable.compareTo()
3. tieBreakOrder() (class name + identityHashCode)

`tieBreakOrder()` is **used only during insertion**, never during lookup.

---

## 5. Tree Bin Example

```
        K3
       /  \
     K1    K5
       \   /
        K2 K4
```

---

## 6. Lookup (`get()`)

Lookup uses `TreeNode.find()` and **does not use tieBreakOrder()**.

Key steps:
- hash comparison
- equals() check (ONLY success condition)
- Comparable optimization
- controlled fallback search

---

## 7. Controlled Fallback Logic

If ordering is ambiguous:
1. Search right subtree
2. If not found, search left subtree

Traversal remains bounded (O(log n)).

---

## 8. Dynamic Lookup Keys

Dynamically created keys work because:
- equals() is checked at every visited node
- identityHashCode of lookup key is irrelevant

---

## 9. Miss Case

Even if all hashes are equal:
- no full traversal
- no linked-list degradation
- bounded RB-tree search

---

## 10. Real Failure Case — Mutable Keys

Changing key fields used in equals/hashCode after insertion breaks HashMap.

---

## 11. Final Mental Model

Insertion:
hash → Comparable → tieBreakOrder → RB-tree

Lookup:
hash → equals → Comparable → fallback

Correctness depends **only on equals()**.

## Even with fallback, HashMap tree-bin lookup remains O(log n) because the Red-Black Tree height bounds traversal, and fallback adds only constant extra work per level without revisiting nodes or scanning entire subtrees.
