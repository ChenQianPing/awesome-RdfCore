# Rdf.Data的使用
- [1.新增操作](#1新增操作)
- [2.获取实体集合](#2获取实体集合)
- [3.分页获取实体的列表](#3分页获取实体的列表)
- [4.获取单个实体](#4获取单个实体)
- [5.更新操作](#5更新操作)
- [6.批量删除操作](#6批量删除操作)
- [7.自定义Sql查询](#7自定义Sql查询)
- [8.多表分页组合条件查询](#8多表分页组合条件查询)
- [9.事务操作](#9事务操作)

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

## 8.多表分页组合条件查询
```
public PageOutput GetPageUser2(GetPageUserInput input, string tenantId, string userId)
{
    /*
        * 五表查询例子 st, sc, st2, st3, st4；
        * 多表分页查询示例
        */
    var where = " 1 = 1";

    if (!string.IsNullOrEmpty(input.Keyword))
        where += $@" AND st.USER_NAME LIKE '%'||'{input.Keyword}'||'%'";

    if (!string.IsNullOrEmpty(input.RoleId))
        where += $@" AND st2.ROLE_ID = '{input.RoleId}'";

    if (string.IsNullOrEmpty(input.OrderBy))
        input.OrderBy = "st.DISP_ORDER";

    if (input.PageIndex == 0)
        input.PageIndex = 1;

    if (input.PageSize == 0)
        input.PageIndex = 10;

    var totalCount = 0;
    var lstResult = _db.Queryable<User, UserRole, Role>((st, sc, st2) => new object[] { JoinType.Inner, st.UserId == sc.UserId, JoinType.Inner, sc.RoleId == st2.RoleId })
        .Where(where)
        .Select((st, sc, st2) => new GetDetailUserOutput
        {
            UserId = st.UserId,
            UserName = st.UserName,
            OrgId = st.OrgId,
            FullName = st.FullName,
            Password = st.Password,
            HashType = st.HashType,
            RndNo = st.RndNo,
            Phone = st.Phone,
            EmailAddress = st.EmailAddress,
            IsEmailConfirmed = st.IsEmailConfirmed,
            EmailConfirmationCode = st.EmailConfirmationCode,
            PasswordResetCode = st.PasswordResetCode,
            LastLoginTime = st.LastLoginTime,
            LoginIp = st.LoginIp,
            UserType = st.UserType,
            IsActive = st.IsActive,
            TenantId = st.TenantId,
            DispOrder = st.DispOrder,
            IsDeleted = st.IsDeleted,
            DeleterUserId = st.DeleterUserId,
            DeletionTime = st.DeletionTime,
            LastModifierUserId = st.LastModifierUserId,
            LastModificationTime = st.LastModificationTime,
            CreatorUserId = st.CreatorUserId,
            CreationTime = st.CreationTime
        })
        .OrderByIF(!string.IsNullOrEmpty(input.OrderBy), input.OrderBy)
        .ToPageList(input.PageIndex, input.PageSize, ref totalCount);

    var output = new PageOutput
    {
        Total = totalCount,
        Records = lstResult
    };

    return output;
}
```

## 9.事务操作
```
public bool BatchDeleteUser(BatchDeleteInput<string> input, string tenantId, string userId)
{
    var result = _db.Ado.UseTran(() =>
    {
        // 这里写你的逻辑
        _db.Deleteable<UserRole>(p => input.Ids.Contains(p.UserId)).EnableDiffLogEvent().ExecuteCommand();
        _db.Deleteable<User>(p => input.Ids.Contains(p.UserId)).EnableDiffLogEvent().ExecuteCommand();
    });
    if (result.IsSuccess)
    {
        // 成功
        return result.IsSuccess;
    }
    else
    {
        throw new CustomException(result.ErrorMessage);
    }            
}
```
