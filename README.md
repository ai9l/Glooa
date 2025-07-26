# Glooa
Zaid
import { sql, relations } from 'drizzle-orm';
import {
  index,
  jsonb,
  pgTable,
  timestamp,
  varchar,
  text,
  integer,
  decimal,
  boolean,
  pgEnum,
} from "drizzle-orm/pg-core";
import { createInsertSchema } from "drizzle-zod";
import { z } from "zod";
// Session storage table (mandatory for Replit Auth)
export const sessions = pgTable(
  "sessions",
  {
    sid: varchar("sid").primaryKey(),
    sess: jsonb("sess").notNull(),
    expire: timestamp("expire").notNull(),
  },
  (table) => [index("IDX_session_expire").on(table.expire)],
);
// User types enum
export const userRoleEnum = pgEnum('user_role', ['student', 'teacher', 'admin']);
// Users table (mandatory for Replit Auth)
export const users = pgTable("users", {
  id: varchar("id").primaryKey().default(sqlgen_random_uuid()),
  email: varchar("email").unique(),
  firstName: varchar("first_name"),
  lastName: varchar("last_name"),
  profileImageUrl: varchar("profile_image_url"),
  role: userRoleEnum("role").default('student'),
  createdAt: timestamp("created_at").defaultNow(),
  updatedAt: timestamp("updated_at").defaultNow(),
});
// Categories table
export const categories = pgTable("categories", {
  id: varchar("id").primaryKey().default(sqlgen_random_uuid()),
  name: varchar("name", { length: 255 }).notNull(),
  nameAr: varchar("name_ar", { length: 255 }).notNull(),
  description: text("description"),
  descriptionAr: text("description_ar"),
  icon: varchar("icon", { length: 100 }),
  color: varchar("color", { length: 7 }).default('#2563eb'),
  createdAt: timestamp("created_at").defaultNow(),
});
// Content types enum
export const contentTypeEnum = pgEnum('content_type', ['summary', 'course', 'exam', 'research']);
// Content items table
export const contentItems = pgTable("content_items", {
  id: varchar("id").primaryKey().default(sqlgen_random_uuid()),
  title: varchar("title", { length: 500 }).notNull(),
  description: text("description"),
  type: contentTypeEnum("type").notNull(),
  categoryId: varchar("category_id").references(() => categories.id),
  authorId: varchar("author_id").references(() => users.id),
  price: decimal("price", { precision: 10, scale: 2 }).notNull(),
  rating: decimal("rating", { precision: 3, scale: 2 }).default('0'),
  ratingCount: integer("rating_count").default(0),
  imageUrl: varchar("image_url"),
  fileUrl: varchar("file_url"),
  isActive: boolean("is_active").default(true),
  isFeatured: boolean("is_featured").default(false),
  createdAt: timestamp("created_at").defaultNow(),
  updatedAt: timestamp("updated_at").defaultNow(),
});
// Cart items table
export const cartItems = pgTable("cart_items", {
  id: varchar("id").primaryKey().default(sqlgen_random_uuid()),
  userId: varchar("user_id").references(() => users.id),
  contentId: varchar("content_id").references(() => contentItems.id),
  quantity: integer("quantity").default(1),
  createdAt: timestamp("created_at").defaultNow(),
});
// Orders table
export const orders = pgTable("orders", {
  id: varchar("id").primaryKey().default(sqlgen_random_uuid()),
  userId: varchar("user_id").references(() => users.id),
  total: decimal("total", { precision: 10, scale: 2 }).notNull(),
  status: varchar("status", { length: 50 }).default('pending'),
  createdAt: timestamp("created_at").defaultNow(),
});
// Order items table
export const orderItems = pgTable("order_items", {
  id: varchar("id").primaryKey().default(sqlgen_random_uuid()),
  orderId: varchar("order_id").references(() => orders.id),
  contentId: varchar("content_id").references(() => contentItems.id),
  price: decimal("price", { precision: 10, scale: 2 }).notNull(),
  quantity: integer("quantity").default(1),
});
// Relations
export const usersRelations = relations(users, ({ many }) => ({
  contentItems: many(contentItems),
  cartItems: many(cartItems),
  orders: many(orders),
}));
export const categoriesRelations = relations(categories, ({ many }) => ({
  contentItems: many(contentItems),
}));
export const contentItemsRelations = relations(contentItems, ({ one, many }) => ({
  category: one(categories, {
    fields: [contentItems.categoryId],
    references: [categories.id],
  }),
  author: one(users, {
    fields: [contentItems.authorId],
    references: [users.id],
  }),
  cartItems: many(cartItems),
  orderItems: many(orderItems),
}));
export const cartItemsRelations = relations(cartItems, ({ one }) => ({
  user: one(users, {
    fields: [cartItems.userId],
    references: [users.id],
  }),
  content: one(contentItems, {
    fields: [cartItems.contentId],
    references: [contentItems.id],
  }),
}));
export const ordersRelations = relations(orders, ({ one, many }) => ({
  user: one(users, {
    fields: [orders.userId],
    references: [users.id],
  }),
  orderItems: many(orderItems),
}));
export const orderItemsRelations = relations(orderItems, ({ one }) => ({
  order: one(orders, {
    fields: [orderItems.orderId],
    references: [orders.id],
  }),
  content: one(contentItems, {
    fields: [orderItems.contentId],
    references: [contentItems.id],
  }),
}));
// Insert schemas
export const insertUserSchema = createInsertSchema(users).omit({
  id: true,
  createdAt: true,
  updatedAt: true,
});
export const insertCategorySchema = createInsertSchema(categories).omit({
  id: true,
  createdAt: true,
});
export const insertContentItemSchema = createInsertSchema(contentItems).omit({
  id: true,
  createdAt: true,
  updatedAt: true,
  rating: true,
  ratingCount: true,
});
export const insertCartItemSchema = createInsertSchema(cartItems).omit({
  id: true,
  createdAt: true,
});
export const insertOrderSchema = createInsertSchema(orders).omit({
  id: true,
  createdAt: true,
});
// Types
export type UpsertUser = typeof users.$inferInsert;
export type User = typeof users.$inferSelect;
export type InsertUser = z.infer<typeof insertUserSchema>;
export type Category = typeof categories.$inferSelect;
export type InsertCategory = z.infer<typeof insertCategorySchema>;
export type ContentItem = typeof contentItems.$inferSelect;
export type InsertContentItem = z.infer<typeof insertContentItemSchema>;
export type CartItem = typeof cartItems.$inferSelect;
export type InsertCartItem = z.infer<typeof insertCartItemSchema>;
export type Order = typeof orders.$inferSelect;
export type InsertOrder = z.infer<typeof insertOrderSchema>;
export type OrderItem = typeof orderItems.$inferSelect;
import {
  users,
  categories,
  contentItems,
  cartItems,
  orders,
  orderItems,
  type User,
  type UpsertUser,
  type Category,
  type InsertCategory,
  type ContentItem,
  type InsertContentItem,
  type CartItem,
  type InsertCartItem,
  type Order,
  type InsertOrder,
  type OrderItem,
} from "@shared/schema";
import { db } from "./db";
import { eq, desc, and, like, inArray } from "drizzle-orm";
export interface IStorage {
  // User operations (mandatory for Replit Auth)
  getUser(id: string): Promise<User | undefined>;
  upsertUser(user: UpsertUser): Promise<User>;
  // Category operations
  getCategories(): Promise<Category[]>;
  createCategory(category: InsertCategory): Promise<Category>;
  // Content operations
  getContentItems(filters?: {
    type?: string;
    categoryId?: string;
    isFeatured?: boolean;
    search?: string;
    limit?: number;
  }): Promise<ContentItem[]>;
  getContentItem(id: string): Promise<ContentItem | undefined>;
  createContentItem(content: InsertContentItem): Promise<ContentItem>;
  updateContentItem(id: string, content: Partial<ContentItem>): Promise<ContentItem>;
  // Cart operations
  getCartItems(userId: string): Promise<CartItem[]>;
  addToCart(cartItem: InsertCartItem): Promise<CartItem>;
  removeFromCart(userId: string, contentId: string): Promise<void>;
  clearCart(userId: string): Promise<void>;
  // Order operations
  createOrder(order: InsertOrder, items: { contentId: string; price: number; quantity: number }[]): Promise<Order>;
  getUserOrders(userId: string): Promise<Order[]>;
}
export class DatabaseStorage implements IStorage {
  // User operations
  async getUser(id: string): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.id, id));
    return user;
  }
  async upsertUser(userData: UpsertUser): Promise<User> {
    const [user] = await db
      .insert(users)
      .values(userData)
      .onConflictDoUpdate({
        target: users.id,
        set: {
          ...userData,
          updatedAt: new Date(),
        },
      })
      .returning();
    return user;
  }
  // Category operations
  async getCategories(): Promise<Category[]> {
    return await db.select().from(categories).orderBy(categories.name);
  }
  async createCategory(category: InsertCategory): Promise<Category> {
    const [newCategory] = await db.insert(categories).values(category).returning();
    return newCategory;
  }
  // Content operations
  async getContentItems(filters: {
    type?: string;
    categoryId?: string;
    isFeatured?: boolean;
    search?: string;
    limit?: number;
  } = {}): Promise<ContentItem[]> {
    let query = db
      .select()
      .from(contentItems)
      .where(eq(contentItems.isActive, true));
    const conditions = [];
    if (filters.type) {
      conditions.push(eq(contentItems.type, filters.type as any));
    }
    if (filters.categoryId) {
      conditions.push(eq(contentItems.categoryId, filters.categoryId));
    }
    if (filters.isFeatured !== undefined) {
      conditions.push(eq(contentItems.isFeatured, filters.isFeatured));
    }
    if (filters.search) {
      conditions.push(like(contentItems.title, %${filters.search}%));
    }
    if (conditions.length > 0) {
      query = query.where(and(...conditions));
    }
    query = query.orderBy(desc(contentItems.createdAt));
    if (filters.limit) {
      query = query.limit(filters.limit);
    }
    return await query;
  }
  async getContentItem(id: string): Promise<ContentItem | undefined> {
    const [item] = await db
      .select()
      .from(contentItems)
      .where(and(eq(contentItems.id, id), eq(contentItems.isActive, true)));
    return item;
  }
  async createContentItem(content: InsertContentItem): Promise<ContentItem> {
    const [newContent] = await db.insert(contentItems).values(content).returning();
    return newContent;
  }
  async updateContentItem(id: string, content: Partial<ContentItem>): Promise<ContentItem> {
    const [updatedContent] = await db
      .update(contentItems)
      .set({ ...content, updatedAt: new Date() })
      .where(eq(contentItems.id, id))
      .returning();
    return updatedContent;
  }
  // Cart operations
  async getCartItems(userId: string): Promise<CartItem[]> {
    return await db
      .select()
      .from(cartItems)
      .where(eq(cartItems.userId, userId));
  }
  async addToCart(cartItem: InsertCartItem): Promise<CartItem> {
    // Check if item already exists in cart
    const [existingItem] = await db
      .select()
      .from(cartItems)
      .where(
        and(
          eq(cartItems.userId, cartItem.userId!),
          eq(cartItems.contentId, cartItem.contentId!)
        )
      );
    if (existingItem) {
      // Update quantity
      const [updatedItem] = await db
        .update(cartItems)
        .set({ quantity: existingItem.quantity + (cartItem.quantity || 1) })
        .where(eq(cartItems.id, existingItem.id))
        .returning();
      return updatedItem;
    } else {
      // Insert new item
      const [newItem] = await db.insert(cartItems).values(cartItem).returning();
      return newItem;
    }
  }
  async removeFromCart(userId: string, contentId: string): Promise<void> {
    await db
      .delete(cartItems)
      .where(
        and(
          eq(cartItems.userId, userId),
          eq(cartItems.contentId, contentId)
        )
      );
  }
  async clearCart(userId: string): Promise<void> {
    await db.delete(cartItems).where(eq(cartItems.userId, userId));
  }
  // Order operations
  async createOrder(
    order: InsertOrder,
    items: { contentId: string; price: number; quantity: number }[]
  ): Promise<Order> {
    const [newOrder] = await db.insert(orders).values(order).returning();
   
    if (items.length > 0) {
      await db.insert(orderItems).values(
        items.map(item => ({
          orderId: newOrder.id,
          contentId: item.contentId,
          price: item.price.toString(),
          quantity: item.quantity,
        }))
      );
    }
    return newOrder;
  }
  async getUserOrders(userId: string): Promise<Order[]> {
    return await db
      .select()
      .from(orders)
      .where(eq(orders.userId, userId))
      .orderBy(desc(orders.createdAt));
  }
}
export const storage = new DatabaseStorage();

