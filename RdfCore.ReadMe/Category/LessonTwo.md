# Rdf.Data��ʹ��
- [1.��������](#1��������)
- [2.��ȡʵ�弯��](#2��ȡʵ�弯��)
- [3.��ҳ��ȡʵ����б�](#3��ҳ��ȡʵ����б�)
- [4.��ȡ����ʵ��](#4��ȡ����ʵ��)
- [5.���²���](#5���²���)
- [6.����ɾ������](#6����ɾ������)
- [7.�Զ���Sql��ѯ](#7�Զ���Sql��ѯ)
- [8.����ҳ���������ѯ](#8����ҳ���������ѯ)
- [9.�������](#9�������)

## 1.��������
```
public async Task<int> CreateRole(CreateRoleInput input, string tenantId, string userId)
{
    if (string.IsNullOrWhiteSpace(input.RoleName))
        throw new CustomException("��ɫ���Ʋ���Ϊ�գ�");

    if (await IsExistAsync(p => p.RoleName == input.RoleName))
        throw new CustomException($@"{input.RoleName}��ɫ�����Ѿ����ڣ�������ʹ��һ���½�ɫ���ƣ�");

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
## 2.��ȡʵ�弯��
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

## 3.��ҳ��ȡʵ����б�
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

## 4.��ȡ����ʵ��
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

## 5.���²���
```
public async Task<int> UpdateRole(UpdateRoleInput input, string tenantId, string userId)
{
    if (await IsExistAsync(p => p.RoleName == input.RoleName && p.RoleId != input.RoleId))
        throw new CustomException($@"{input.RoleName}��ɫ�����Ѵ����������ظ�������");

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

## 6.����ɾ������
```
public async Task<int> BatchDeleteRole(BatchDeleteInput<string> input, string tenantId, string userId)
{
    return await DeleteAsync(p => input.Ids.Contains(p.RoleId));
}
```

## 7.�Զ���Sql��ѯ
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

## 8.����ҳ���������ѯ
```
public PageOutput GetPageUser2(GetPageUserInput input, string tenantId, string userId)
{
    /*
        * ����ѯ���� st, sc, st2, st3, st4��
        * ����ҳ��ѯʾ��
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

## 9.�������
```
public bool BatchDeleteUser(BatchDeleteInput<string> input, string tenantId, string userId)
{
    var result = _db.Ado.UseTran(() =>
    {
        // ����д����߼�
        _db.Deleteable<UserRole>(p => input.Ids.Contains(p.UserId)).EnableDiffLogEvent().ExecuteCommand();
        _db.Deleteable<User>(p => input.Ids.Contains(p.UserId)).EnableDiffLogEvent().ExecuteCommand();
    });
    if (result.IsSuccess)
    {
        // �ɹ�
        return result.IsSuccess;
    }
    else
    {
        throw new CustomException(result.ErrorMessage);
    }            
}
```
