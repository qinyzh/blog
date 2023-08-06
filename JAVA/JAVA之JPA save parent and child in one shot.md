# JAVA之JAP save parent and child in one shot
为了能够在保存parent entity的同时保存它的children,有一些规则需要遵守：
- cascade type使用CascadeType.PERSIST（type使用CascadeType.ALL 也可以）。
- 应该在两边设置正确的双向关系（例如：parent包含其集合字段的所有children,每个child都有对parent的引用）。
- 在transaction的范围内进行数据操作，不允许采用自动提交的模式。
- 只有parent entity会被手动保存(因为有cascade模式，所以children会被自动保存)。
- Hibernate先插入parent records, 然后插入child records,然后更新child records的所有外键。
- 优先使用fetch = FetchType.Lazy
- 最好使用辅助方法来设置辅助关系。
  ```
  //Cart（Parent）应该包含这个方法
  public void addCartItem(CartItem item){
    cartItems.add(item); // CartItem(Child)
    item.setCart(this);
}
  ```

例子：Company和Department是父子关系，Department内部还包含父子关系，

 ```
 public class Company{
    @Id
    @Column(name = "company_id")
    @SequenceGenerator(name="company_id_seq", schema="XXX", sequenceName="table_name_company_id_seq",AllocationsSize=1)
    @GeneratedValue(strategy= GenerationType.SEQUENCE, generator = "company_id_seq")
    proteded Long id; //自动生成id，增量为1
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY, mappedBy = "company")
    private List<Department> department = new ArrayList<>(); //最好初始化为private类型，由于Hibernate使用自己的List实现，永远不应该直接通过setter来改变它。
 }
 ```
 ```
public class Department{
    @Id
    @Column(name = "department_id")
    @SequenceGenerator(name="department_id_seq", schema="XXX", sequenceName="table_name_department_id_seq",AllocationsSize=1)
    @GeneratedValue(strategy= GenerationType.SEQUENCE, generator = "department_id_seq")
    proteded Long id; //自动生成id，增量为1

    @Column(name = "parent_department_id", insertable = false, updatable = false)
    private Long parentDepartmentId;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "company_id")
    private Company company;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_department_id")
    @JsonIgnore
    private Deparement parentDep;

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY, mappedBy = "parentDep")
    private Set<Department> childDeps = new LinkedHashSet<>();

    @Override
    public boolean equals(Object o){
        if(this == 0) return true;
        if(o == null || getClass() !=o.getClass()) return false;
        Department that = （Department）o;
        if(that.id == null|| id==null){
            return Object.equals(XX,that.XX)
        }
        return Objects.equals(id, that.id);
        //如果不用id去判断两个对象是否相等，需要修改equals里面的规则,因为id是自动生成的，刚保存的时候id都是null，会导致保存不进去。
    }
 
 ```
 ```
 //save Department entity
 for(DepartmentDto depDto: company.getDepartment()){
    //save department
    Department dep = new Department(depDto);
    dep.setId(null);
    //save sub department 
    for(DepartmentDto subDepDto:depDto.getChildDeps()){
       Department subDep = new Department(subDepDto); 
       subDep.setParentDep(dep);
       dep.getChildDeps().add(subDep);
    }
    //save sub end
    company.getDepartment().add(dep);
 }
 ```

 Ref:https://stackoverflow.com/questions/53647672/how-to-save-parent-and-child-in-one-shot-jpa-hibernate
