# README

Ansible Role to create a RootCA + Intermediate + End-entity cert

```bash
┌──────────────────────┐             ┌──────────────────────┐               ┌──────────────────────┐
│                      │    Signed   │                      │     Signed    │                      │
│                      │      by     │                      │       by      │                      │
│        RootCA        │◄────────────┤    IntemediateCA     │◄──────────────┤       End-Entity     │
│                      │             │                      │               │       Certificate    │
│                      │             │                      │               │                      │
└───┬──────────────▲───┘             └──────────────────────┘               └──────────────────────┘
    │              │
    │              │
    │    Signed    │
    └──────────────┘
```

# ToDo

- Multiple Intermediate
