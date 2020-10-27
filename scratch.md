# Show Access-List

```
show access-list
```

# Remove ACL From Interface
_Removes outgoing access group 11 (ACL 11 from the Serial0/0/0) Interface_
```
conf t
int s0/0/0
no ip access-group 11 out
end
```

# Remove ACL
_Removes ACL number 10_
```
no access-list 10
```

