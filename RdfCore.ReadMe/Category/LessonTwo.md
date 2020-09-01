# Rdf.Data的使用
- [1.新增操作](#1新增操作)
- [2.获取实体集合](#2获取实体集合)
- [3.分页获取实体的列表](#3分页获取实体的列表)
- [4.获取单个实体](#4获取单个实体)
- [5.更新操作](#5更新操作)
- [6.批量删除操作](#6批量删除操作)
- [7.自定义Sql查询](#7自定义Sql查询)

## 1.新增操作
```
public async Task<int> CreateRole(CreateRoleInput input, string tenantId, string userId)
{
    if (string.IsNullOrWhiteSpace(input.RoleName))
        throw new CustomException("角色名称不能为空！");

    if (await IsExistAsync(p => p.RoleName == input.RoleName))
        throw new CustomException($@"{input.RoleName}角色名称已经存在，请重新使用一个新角色名称！");

    var entity = new Role
    {
        RoleId = GuidHelper.GenerateNewId(),
        RoleName = input.RoleName,
        RoleType = input.RoleType,
        RoleDesc = input.RoleDesc,
        IsStatic = input.IsStatic,
        IsDefault = input.IsDefault,
        DispOrder = input.DispOrder,
        TenantId = tenantId,
        IsDeleted = 0,
        CreatorUserId = userId,
        CreationTime = System.DateTime.Now
    };

    return await InsertAsync(entity);
}
```
## 2.获取实体集合
```
public async Task<List<GetDetailRoleOutput>> GetAllRole(string tenantId, string userId)
{
    var query = GetAll().Select(o => new GetDetailRoleOutput
    {
        RoleId = o.RoleId,
        RoleName = o.RoleName,
        RoleType = o.RoleType,
        RoleDesc = o.RoleDesc,
        IsStatic = o.IsStatic,
        IsDefault = o.IsDefault,
        DispOrder = o.DispOrder
    }).OrderBy(p => p.DispOrder);

    return await query.ToListAsync();
}
```

## 3.分页获取实体的列表
```
public async Task<PageOutput> GetPageRole(GetPageRoleInput input, string tenantId, string userId)
{
    if (string.IsNullOrEmpty(input.OrderBy))
        input.OrderBy = "DISP_ORDER";

    Expression<Func<Role, bool>> where = (it => 1 == 1);

    if (!string.IsNullOrEmpty(input.Keyword))
    {
        Expression<Func<Role, bool>> exp1 = (p => p.RoleName.Contains(input.Keyword));
        where = ExpressionHelp.And(where, exp1);
    }

    return await GetPageAsync(where, input.OrderBy, input.PageIndex, input.PageSize);
}
```

## 4.获取单个实体
```
public async Task<GetDetailRoleOutput> GetDetailRole(GetDetailInput<string> input, string tenantId, string userId)
{
    var o = await GetByIdAsync(input.Id);

    var output = new GetDetailRoleOutput
    {
        RoleId = o.RoleId,
        RoleName = o.RoleName,
        RoleType = o.RoleType,
        RoleDesc = o.RoleDesc,
        IsStatic = o.IsStatic,
        IsDefault = o.IsDefault,
        DispOrder = o.DispOrder,
        TenantId = o.TenantId,
        IsDeleted = o.IsDeleted,
        DeleterUserId = o.DeleterUserId,
        DeletionTime = o.DeletionTime,
        LastModifierUserId = o.LastModifierUserId,
        LastModificationTime = o.LastModificationTime,
        CreatorUserId = o.CreatorUserId,
        CreationTime = o.CreationTime
    };
    return output;
}
```

## 5.更新操作
```
public async Task<int> UpdateRole(UpdateRoleInput input, string tenantId, string userId)
{
    if (await IsExistAsync(p => p.RoleName == input.RoleName && p.RoleId != input.RoleId))
        throw new CustomException($@"{input.RoleName}角色名称已创建，不能重复创建！");

    var entity = new Role
    {
        RoleId = input.RoleId,
        RoleName = input.RoleName,
        RoleType = input.RoleType,
        RoleDesc = input.RoleDesc,
        IsStatic = input.IsStatic,
        IsDefault = input.IsDefault,
        DispOrder = input.DispOrder,
        TenantId = tenantId,
        IsDeleted = input.IsDeleted == null ? 0 : input.IsDeleted,
        LastModifierUserId = userId,
        LastModificationTime = System.DateTime.Now
    };

    return await UpdateAsync(entity);
}
```

## 6.批量删除操作
```
public async Task<int> BatchDeleteRole(BatchDeleteInput<string> input, string tenantId, string userId)
{
    return await DeleteAsync(p => input.Ids.Contains(p.RoleId));
}
```

## 7.自定义Sql查询
```
public async Task<IEnumerable<GetDetailRoleOutput>> GetAllRole2(string tenantId, string userId)
{
    #region sqlQuery
    var sqlQuery = $@"SELECT T1.ROLE_ID RoleId
            ,T1.ROLE_NAME RoleName
            ,T1.ROLE_TYPE RoleType
            ,T1.ROLE_DESC RoleDesc
            ,T1.IS_STATIC IsStatic
            ,T1.IS_DEFAULT IsDefault
            ,T1.DISP_ORDER DispOrder
        FROM ACT_ROLE T1
        ORDER BY T1.DISP_ORDER";
    #endregion

    return await SqlQueryAsync<GetDetailRoleOutput>(sqlQuery);
}
```