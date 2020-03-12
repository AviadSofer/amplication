# Migration `20200311132301-change-user-to-account`

This migration has been generated by Yuval Hazaz at 3/11/2020, 1:23:01 PM.
You can check out the [state of the schema](./schema.prisma) after the migration.

## Database Steps

```sql
CREATE TYPE "EnumDataType" AS ENUM ('Text', 'AutoNumber', 'WholeNumber', 'TimeZone', 'Language', 'DateAndTime', 'Currancy', 'DecimalNumber', 'File', 'Image', 'Lookup', 'CustomEntity', 'OptionSet', 'Boolean', 'Color', 'Guid', 'Time', 'CalculatedField', 'RollupField');

CREATE TYPE "EnumRoleLevel" AS ENUM ('Organization', 'Project');

CREATE TABLE "public"."Organization" (
    "address" text  NOT NULL DEFAULT '',
    "createdAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    "defaultTimeZone" text  NOT NULL DEFAULT '',
    "id" text  NOT NULL ,
    "name" text  NOT NULL DEFAULT '',
    "updatedAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    PRIMARY KEY ("id")
) 

CREATE TABLE "public"."Project" (
    "createdAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    "id" text  NOT NULL ,
    "name" text  NOT NULL DEFAULT '',
    "organization" text  NOT NULL ,
    "updatedAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    PRIMARY KEY ("id")
) 

CREATE TABLE "public"."Entity" (
    "allowFeedback" boolean  NOT NULL DEFAULT false,
    "createdAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    "description" text  NOT NULL DEFAULT '',
    "displayName" text  NOT NULL DEFAULT '',
    "id" text  NOT NULL ,
    "isPersistent" boolean  NOT NULL DEFAULT false,
    "name" text  NOT NULL DEFAULT '',
    "pluralDisplayName" text  NOT NULL DEFAULT '',
    "primaryField" text  NOT NULL DEFAULT '',
    "projects" text  NOT NULL ,
    "updatedAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    PRIMARY KEY ("id")
) 

CREATE TABLE "public"."EntityField" (
    "createdAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    "dataType" "EnumDataType" NOT NULL DEFAULT 'Text',
    "dataTypeProperties" text  NOT NULL DEFAULT '',
    "description" text  NOT NULL DEFAULT '',
    "displayName" text  NOT NULL DEFAULT '',
    "entity" text  NOT NULL ,
    "id" text  NOT NULL ,
    "name" text  NOT NULL DEFAULT '',
    "properties" text  NOT NULL DEFAULT '',
    "required" boolean  NOT NULL DEFAULT false,
    "searchable" boolean  NOT NULL DEFAULT false,
    "updatedAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    PRIMARY KEY ("id")
) 

CREATE TABLE "public"."Account" (
    "createdAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    "email" text  NOT NULL DEFAULT '',
    "firstname" text  NOT NULL DEFAULT '',
    "id" text  NOT NULL ,
    "lastName" text  NOT NULL DEFAULT '',
    "password" text  NOT NULL DEFAULT '',
    "updatedAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    PRIMARY KEY ("id")
) 

CREATE TABLE "public"."AccountUser" (
    "account" text  NOT NULL ,
    "createdAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    "id" text  NOT NULL ,
    "organization" text  NOT NULL ,
    "updatedAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    PRIMARY KEY ("id")
) 

CREATE TABLE "public"."UserRole" (
    "accountUser" text  NOT NULL ,
    "createEntity" boolean  NOT NULL DEFAULT false,
    "createFlow" boolean  NOT NULL DEFAULT false,
    "createUi" boolean  NOT NULL DEFAULT false,
    "createWebServices" boolean  NOT NULL DEFAULT false,
    "createdAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    "id" text  NOT NULL ,
    "roleLevel" "EnumRoleLevel" NOT NULL DEFAULT 'Organization',
    "updatedAt" timestamp(3)  NOT NULL DEFAULT '1970-01-01 00:00:00',
    PRIMARY KEY ("id")
) 

ALTER TABLE "public"."Project" ADD FOREIGN KEY ("organization")REFERENCES "public"."Organization"("id") ON DELETE RESTRICT  ON UPDATE CASCADE

ALTER TABLE "public"."Entity" ADD FOREIGN KEY ("projects")REFERENCES "public"."Project"("id") ON DELETE RESTRICT  ON UPDATE CASCADE

ALTER TABLE "public"."EntityField" ADD FOREIGN KEY ("entity")REFERENCES "public"."Entity"("id") ON DELETE RESTRICT  ON UPDATE CASCADE

ALTER TABLE "public"."AccountUser" ADD FOREIGN KEY ("organization")REFERENCES "public"."Organization"("id") ON DELETE RESTRICT  ON UPDATE CASCADE

ALTER TABLE "public"."AccountUser" ADD FOREIGN KEY ("account")REFERENCES "public"."Account"("id") ON DELETE RESTRICT  ON UPDATE CASCADE

ALTER TABLE "public"."UserRole" ADD FOREIGN KEY ("accountUser")REFERENCES "public"."AccountUser"("id") ON DELETE RESTRICT  ON UPDATE CASCADE

DROP TABLE "public"."Post";

DROP TABLE "public"."User";
```

## Changes

```diff
diff --git schema.prisma schema.prisma
migration 20200302164202-add-address..20200311132301-change-user-to-account
--- datamodel.dml
+++ datamodel.dml
@@ -1,41 +1,25 @@
 datasource sqlite {
   provider = "sqlite"
-  url = "***"
+  url      = "file:./dev.db"
   enabled  = false
 }
 datasource db {
   provider = "postgresql"
-  url = "***"
+  url      = env("POSTGRESQL_URL")
   enabled  = true
 }
 generator client {
   provider = "prisma-client-js"
 }
-model User {
-  id        String   @default(cuid()) @id
-  createdAt DateTime @default(now())
-  updatedAt DateTime @updatedAt
-  email     String   @unique
-  password  String
-  firstname String?
-  lastname  String?
-  posts     Post[]
-  role      Role
+generator typegraphql {
+  provider = "node_modules/typegraphql-prisma-nestjs/generator.js"
+  output   = "./dal"
 }
-model Post {
-  id        String   @default(cuid()) @id
-  createdAt DateTime @default(now())
-  updatedAt DateTime @updatedAt
-  published Boolean
-  title     String
-  content   String?
-  author    User?
-}
 enum Role {
   ADMIN
   USER
@@ -47,21 +31,19 @@
   updatedAt DateTime @updatedAt
   name  String
   defaultTimeZone String
   address String
-  // projects  Project[]
-  // accountUsers AccountUser[]
+  projects  Project[]
+  accountUsers AccountUser[]
 }
 model Project {
   id    String   @default(cuid()) @id
   createdAt DateTime @default(now())
   updatedAt DateTime @updatedAt
   organization Organization
   name  String 
-  defaultTimeZone String
-  entity  Entity[]
-  
+  entity  Entity[] 
   //@@index([organization])
 }
 model Entity {
```

