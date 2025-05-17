# Fronsciers Complete Code Overview

This document provides a comprehensive overview of the most important code files in the Fronsciers platform. This compilation is designed to help GitHub users understand the architecture and implementation of our decentralized academic publishing platform.

## Table of Contents
1. [Database Schema](#database-schema)
2. [Backend API Routes](#backend-api-routes)
3. [Storage Layer](#storage-layer)
4. [Solana Smart Contracts](#solana-smart-contracts)
5. [Frontend Components](#frontend-components)
6. [Utility Functions](#utility-functions)

---

## Database Schema
File: `shared/schema.ts`

```typescript
import { relations } from "drizzle-orm";
import {
  pgTable,
  serial,
  text,
  integer,
  timestamp,
  boolean,
  json,
  pgEnum,
} from "drizzle-orm/pg-core";
import { createInsertSchema } from "drizzle-zod";
import { z } from "zod";

// Enum for DOCI types
export const doiTypeEnum = pgEnum("doi_type", [
  "article",
  "preprint",
  "chapter",
  "dataset",
  "software",
  "whitepaper",
]);

// Enum for badge types
export const badgeTypeEnum = pgEnum("badge_type", [
  "bronze",
  "silver",
  "gold",
  "platinum",
]);

// Users table
export const users = pgTable("users", {
  id: serial("id").primaryKey(),
  username: text("username").notNull().unique(),
  email: text("email").notNull().unique(),
  passwordHash: text("password_hash").notNull(),
  fullName: text("full_name").notNull(),
  institution: text("institution"),
  country: text("country"),
  walletAddress: text("wallet_address").unique(),
  profileImage: text("profile_image"),
  bio: text("bio"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  stripeCustomerId: text("stripe_customer_id"),
  stripeSubscriptionId: text("stripe_subscription_id"),
  subscriptionStatus: text("subscription_status"),
  subscriptionTier: text("subscription_tier"),
  subscriptionExpiry: timestamp("subscription_expiry"),
});

// Researcher DOCIs table
export const researcherDoci = pgTable("researcher_doci", {
  id: serial("id").primaryKey(),
  prefix: text("prefix").notNull().default("10.FRONS-R"),
  suffix: text("suffix").notNull().unique(),
  userId: integer("user_id")
    .references(() => users.id)
    .notNull(),
  registrationDate: timestamp("registration_date").defaultNow().notNull(),
  metadataUri: text("metadata_uri"),
  txHash: text("tx_hash"),
  txTimestamp: timestamp("tx_timestamp"),
  issuingAuthority: text("issuing_authority").default("Fronsciers"),
});

// DOIs table
export const dois = pgTable("dois", {
  id: serial("id").primaryKey(),
  prefix: text("prefix").notNull(),
  suffix: text("suffix").notNull(),
  title: text("title").notNull(),
  authors: text("authors").notNull(),
  abstract: text("abstract"),
  type: doiTypeEnum("type").notNull().default("article"),
  status: text("status").notNull().default("published"),
  version: integer("version").notNull().default(1),
  sourceUri: text("source_uri"),
  metadataUri: text("metadata_uri"),
  txHash: text("tx_hash"),
  txTimestamp: timestamp("tx_timestamp"),
  userId: integer("user_id").references(() => users.id),
});

// DOI resolutions table
export const doiResolutions = pgTable("doi_resolutions", {
  id: serial("id").primaryKey(),
  doiId: integer("doi_id")
    .references(() => dois.id)
    .notNull(),
  timestamp: timestamp("timestamp").defaultNow().notNull(),
  referrer: text("referrer"),
});

// Transactions table
export const transactions = pgTable("transactions", {
  id: serial("id").primaryKey(),
  txHash: text("tx_hash").notNull(),
  txType: text("tx_type").notNull(),
  userId: integer("user_id").references(() => users.id),
  doiId: integer("doi_id").references(() => dois.id),
  amount: text("amount"),
  currency: text("currency"),
  status: text("status").notNull(),
  timestamp: timestamp("timestamp").defaultNow().notNull(),
  metadata: json("metadata"),
});

// Badges table (SPL Tokens)
export const badges = pgTable("badges", {
  id: serial("id").primaryKey(),
  type: badgeTypeEnum("type").notNull(),
  name: text("name").notNull(),
  description: text("description"),
  tokenAddress: text("token_address"),
  userId: integer("user_id").references(() => users.id),
  doiId: integer("doi_id").references(() => dois.id),
  issueDate: timestamp("issue_date").defaultNow().notNull(),
  metadata: json("metadata"),
});

// Submissions table
export const submissions = pgTable("submissions", {
  id: serial("id").primaryKey(),
  submissionId: text("submission_id").notNull().unique(),
  title: text("title").notNull(),
  abstract: text("abstract").notNull(),
  authorId: text("author_id").notNull(),
  coauthors: json("coauthors"),
  contentIpfsHash: text("content_ipfs_hash").notNull(),
  metadataIpfsHash: text("metadata_ipfs_hash"),
  status: text("status").notNull().default("pending"),
  submissionDate: timestamp("submission_date").defaultNow().notNull(),
  lastUpdated: timestamp("last_updated").defaultNow().notNull(),
  escrowAddress: text("escrow_address"),
  escrowSignature: text("escrow_signature"),
  reviewCount: integer("review_count").default(0),
  reviewsRequired: integer("reviews_required").default(3),
});

// Reviewers table
export const reviewers = pgTable("reviewers", {
  id: serial("id").primaryKey(),
  reviewerId: text("reviewer_id").notNull().unique(),
  name: text("name").notNull(),
  institution: text("institution"),
  expertise: json("expertise"),
  walletAddress: text("wallet_address").unique(),
  totalReviews: integer("total_reviews").default(0),
  reputation: integer("reputation").default(0),
  availableForReview: boolean("available_for_review").default(true),
  registrationDate: timestamp("registration_date").defaultNow().notNull(),
});

// Reviews table
export const reviews = pgTable("reviews", {
  id: serial("id").primaryKey(),
  reviewId: text("review_id").notNull().unique(),
  submissionId: text("submission_id")
    .notNull()
    .references(() => submissions.submissionId),
  reviewerId: text("reviewer_id")
    .notNull()
    .references(() => reviewers.reviewerId),
  content: text("content").notNull(),
  rating: integer("rating").notNull(),
  recommendation: text("recommendation").notNull(),
  status: text("status").notNull().default("submitted"),
  submissionDate: timestamp("submission_date").defaultNow().notNull(),
  ipfsHash: text("ipfs_hash"),
  onChainSignature: text("on_chain_signature"),
  conflictOfInterest: boolean("conflict_of_interest").default(false),
});

// Citations table
export const citations = pgTable("citations", {
  id: serial("id").primaryKey(),
  citingDoiId: text("citing_doi_id").notNull(),
  citedDoiId: text("cited_doi_id").notNull(),
  context: text("context"),
  page: text("page"),
  timestamp: timestamp("timestamp").defaultNow().notNull(),
  verified: boolean("verified").default(false),
});

// Relations
export const userRelations = relations(users, ({ many }) => ({
  dois: many(dois),
  badges: many(badges),
  transactions: many(transactions),
}));

export const doiRelations = relations(dois, ({ one, many }) => ({
  creator: one(users, {
    fields: [dois.userId],
    references: [users.id],
  }),
  resolutions: many(doiResolutions),
  badges: many(badges),
  transactions: many(transactions),
}));

export const badgeRelations = relations(badges, ({ one }) => ({
  user: one(users, {
    fields: [badges.userId],
    references: [users.id],
  }),
  doi: one(dois, {
    fields: [badges.doiId],
    references: [dois.id],
  }),
}));

export const doiResolutionRelations = relations(doiResolutions, ({ one }) => ({
  doi: one(dois, {
    fields: [doiResolutions.doiId],
    references: [dois.id],
  }),
}));

export const transactionRelations = relations(transactions, ({ one }) => ({
  user: one(users, {
    fields: [transactions.userId],
    references: [users.id],
  }),
  doi: one(dois, {
    fields: [transactions.doiId],
    references: [dois.id],
  }),
}));

export const researcherDociRelations = relations(researcherDoci, ({ one }) => ({
  user: one(users, {
    fields: [researcherDoci.userId],
    references: [users.id],
  }),
}));

// Zod schemas for validation
export const insertUserSchema = createInsertSchema(users).omit({
  id: true,
  createdAt: true,
});

export const insertDoiSchema = createInsertSchema(dois).omit({
  id: true,
});

export const insertResearcherDociSchema = createInsertSchema(researcherDoci).omit({
  id: true,
  registrationDate: true,
});

export const insertBadgeSchema = createInsertSchema(badges).omit({
  id: true,
  issueDate: true,
});

export const insertTransactionSchema = createInsertSchema(transactions).omit({
  id: true,
  timestamp: true,
});

export const insertDoiResolutionSchema = createInsertSchema(doiResolutions).omit({
  id: true,
});

export const insertSubmissionSchema = createInsertSchema(submissions).omit({
  id: true,
  submissionDate: true,
  lastUpdated: true,
});

export const insertReviewerSchema = createInsertSchema(reviewers).omit({
  id: true,
  registrationDate: true,
});

export const insertReviewSchema = createInsertSchema(reviews).omit({
  id: true,
  submissionDate: true,
});

export const insertCitationSchema = createInsertSchema(citations).omit({
  id: true,
  timestamp: true,
});

// Types
export type User = typeof users.$inferSelect;
export type InsertUser = z.infer<typeof insertUserSchema>;

export type Doi = typeof dois.$inferSelect;
export type InsertDoi = z.infer<typeof insertDoiSchema>;

export type ResearcherDoci = typeof researcherDoci.$inferSelect;
export type InsertResearcherDoci = z.infer<typeof insertResearcherDociSchema>;

export type Badge = typeof badges.$inferSelect;
export type InsertBadge = z.infer<typeof insertBadgeSchema>;
export type BadgeType = "bronze" | "silver" | "gold" | "platinum";

export type Transaction = typeof transactions.$inferSelect;
export type InsertTransaction = z.infer<typeof insertTransactionSchema>;

export type DoiResolution = typeof doiResolutions.$inferSelect;
export type InsertDoiResolution = z.infer<typeof insertDoiResolutionSchema>;

export type Submission = typeof submissions.$inferSelect;
export type InsertSubmission = z.infer<typeof insertSubmissionSchema>;

export type Reviewer = typeof reviewers.$inferSelect;
export type InsertReviewer = z.infer<typeof insertReviewerSchema>;

export type Review = typeof reviews.$inferSelect;
export type InsertReview = z.infer<typeof insertReviewSchema>;

export type Citation = typeof citations.$inferSelect;
export type InsertCitation = z.infer<typeof insertCitationSchema>;
```

## Backend API Routes
File: `server/routes.ts`

```typescript
import { Express, Request, Response } from "express";
import { createServer, Server } from "http";
import { WebSocketServer } from "ws";
import { storage } from "./storage";
import { z } from "zod";
import {
  insertDoiSchema,
  insertSubmissionSchema,
  insertReviewSchema,
  insertBadgeSchema,
} from "@shared/schema";
import { getAuthUser } from "./middlewares/auth";

export async function registerRoutes(app: Express): Promise<Server> {
  // DOCI resolution endpoint
  app.get("/api/resolve/:prefix/:suffix", async (req: Request, res: Response) => {
    try {
      const { prefix, suffix } = req.params;
      const fullDoi = `${prefix}/${suffix}`;
      
      // First try to resolve as publication DOCI
      const doi = await storage.getDoiByFullDoi(fullDoi);
      if (doi) {
        await storage.recordDoiResolution({
          doiId: doi.id,
          timestamp: new Date(),
          referrer: req.headers.referer || null
        });
        
        return res.json({
          type: "publication",
          exists: true,
          data: doi
        });
      }
      
      // Try to resolve as researcher DOCI
      if (prefix === "10.FRONS-R") {
        const researcher = await storage.getResearcherDociByFullDoci(fullDoi);
        if (researcher) {
          return res.json({
            type: "researcher",
            exists: true,
            data: researcher
          });
        }
      }
      
      throw new Error("DOCI not found");
    } catch (error) {
      console.error("Error resolving DOCI:", error);
      res.status(500).json({
        exists: false,
        message: error.message || "Failed to resolve DOCI"
      });
    }
  });

  // Get all DOIs
  app.get("/api/dois", async (req: Request, res: Response) => {
    try {
      const limit = req.query.limit ? parseInt(req.query.limit as string) : 10;
      const dois = await storage.getRecentDois(limit);
      res.json(dois);
    } catch (error) {
      console.error("Error fetching DOIs:", error);
      res.status(500).json({ 
        message: "Failed to fetch DOIs" 
      });
    }
  });

  // Get DOI by ID
  app.get("/api/dois/:id", async (req: Request, res: Response) => {
    try {
      const id = parseInt(req.params.id);
      const doi = await storage.getDoi(id);
      
      if (!doi) {
        return res.status(404).json({ 
          message: "DOI not found" 
        });
      }
      
      res.json(doi);
    } catch (error) {
      console.error("Error fetching DOI:", error);
      res.status(500).json({ 
        message: "Failed to fetch DOI",
        error: error instanceof Error ? error.message : "Unknown error"
      });
    }
  });

  // Create new DOI
  app.post("/api/dois", async (req: Request, res: Response) => {
    try {
      const user = getAuthUser(req);
      if (!user) {
        return res.status(401).json({ message: "Authentication required" });
      }
      
      const validatedData = insertDoiSchema.parse(req.body);
      const doi = await storage.createDoi({
        ...validatedData,
        userId: user.id
      });
      
      res.status(201).json(doi);
    } catch (error) {
      console.error("Error creating DOI:", error);
      if (error instanceof z.ZodError) {
        return res.status(400).json({ 
          message: "Validation error", 
          errors: error.errors 
        });
      }
      res.status(500).json({ 
        message: "Failed to create DOI" 
      });
    }
  });

  // Update DOI
  app.patch("/api/dois/:id", async (req: Request, res: Response) => {
    try {
      const user = getAuthUser(req);
      if (!user) {
        return res.status(401).json({ message: "Authentication required" });
      }
      
      const id = parseInt(req.params.id);
      const doi = await storage.getDoi(id);
      
      if (!doi) {
        return res.status(404).json({ 
          message: "DOI not found" 
        });
      }
      
      if (doi.userId !== user.id) {
        return res.status(403).json({ 
          message: "You don't have permission to update this DOI" 
        });
      }
      
      const updateData = req.body;
      // Prevent changing critical fields
      delete updateData.id;
      delete updateData.prefix;
      delete updateData.suffix;
      delete updateData.userId;
      
      const updatedDoi = await storage.updateDoi(id, updateData);
      res.json(updatedDoi);
    } catch (error) {
      console.error("Error updating DOI:", error);
      res.status(500).json({ 
        message: "Failed to update DOI" 
      });
    }
  });

  // Get all transactions
  app.get("/api/transactions", async (req: Request, res: Response) => {
    try {
      const limit = req.query.limit ? parseInt(req.query.limit as string) : 10;
      const transactions = await storage.getRecentTransactions(limit);
      res.json(transactions);
    } catch (error) {
      console.error("Error fetching transactions:", error);
      res.status(500).json({ 
        message: "Failed to fetch transactions" 
      });
    }
  });

  // Get transactions by DOI
  app.get("/api/dois/:doiId/transactions", async (req: Request, res: Response) => {
    try {
      const doiId = parseInt(req.params.doiId);
      const transactions = await storage.getTransactionsByDoi(doiId);
      res.json(transactions);
    } catch (error) {
      console.error("Error fetching transactions for DOI:", error);
      res.status(500).json({ 
        message: "Failed to fetch transactions for DOI" 
      });
    }
  });

  // Get transactions by user
  app.get("/api/users/:userId/transactions", async (req: Request, res: Response) => {
    try {
      const userId = parseInt(req.params.userId);
      const transactions = await storage.getTransactionsByUser(userId);
      res.json(transactions);
    } catch (error) {
      console.error("Error fetching transactions for user:", error);
      res.status(500).json({ 
        message: "Failed to fetch transactions for user" 
      });
    }
  });

  // Get platform statistics
  app.get("/api/stats", async (req: Request, res: Response) => {
    try {
      const doiStats = await storage.getDoiStats();
      const badgeStats = await storage.getBadgeStats();
      
      res.json({
        ...doiStats,
        ...badgeStats
      });
    } catch (error) {
      console.error("Error fetching stats:", error);
      res.status(500).json({ 
        message: "Failed to fetch platform statistics" 
      });
    }
  });

  // Get resolution analytics by timeframe
  app.get("/api/analytics/resolutions/:timeframe", async (req: Request, res: Response) => {
    try {
      const timeframe = req.params.timeframe as 'day' | 'week' | 'month' | 'year';
      const stats = await storage.getDoiResolutionsByTimeframe(timeframe);
      res.json(stats);
    } catch (error) {
      console.error("Error fetching resolution analytics:", error);
      res.status(500).json({ 
        message: "Failed to fetch resolution analytics" 
      });
    }
  });

  // Get DOIs by type
  app.get("/api/analytics/dois-by-type", async (req: Request, res: Response) => {
    try {
      const stats = await storage.getDoisByType();
      res.json(stats);
    } catch (error) {
      console.error("Error fetching DOI type analytics:", error);
      res.status(500).json({ 
        message: "Failed to fetch DOI type analytics" 
      });
    }
  });

  // Get top DOIs by resolution count
  app.get("/api/analytics/top-dois", async (req: Request, res: Response) => {
    try {
      const limit = req.query.limit ? parseInt(req.query.limit as string) : 5;
      const stats = await storage.getTopDoisByResolution(limit);
      res.json(stats);
    } catch (error) {
      console.error("Error fetching top DOIs:", error);
      res.status(500).json({ 
        message: "Failed to fetch top DOIs" 
      });
    }
  });

  // Get citation statistics
  app.get("/api/analytics/citations", async (req: Request, res: Response) => {
    try {
      const stats = await storage.getCitationStats();
      res.json(stats);
    } catch (error) {
      console.error("Error fetching citation statistics:", error);
      res.status(500).json({ 
        message: "Failed to fetch citation statistics" 
      });
    }
  });

  // Badge endpoints
  app.get("/api/badges", async (req: Request, res: Response) => {
    try {
      const limit = req.query.limit ? parseInt(req.query.limit as string) : 10;
      const badges = await storage.getRecentBadges(limit);
      res.json(badges);
    } catch (error) {
      console.error("Error fetching badges:", error);
      res.status(500).json({ 
        message: "Failed to fetch badges" 
      });
    }
  });

  app.get("/api/badges/stats", async (req: Request, res: Response) => {
    try {
      const stats = await storage.getBadgeStats();
      res.json(stats);
    } catch (error) {
      console.error("Error fetching badge stats:", error);
      res.status(500).json({ 
        message: "Failed to fetch badge statistics" 
      });
    }
  });

  app.get("/api/badges/:id", async (req: Request, res: Response) => {
    try {
      const id = parseInt(req.params.id);
      const badge = await storage.getBadge(id);
      
      if (!badge) {
        return res.status(404).json({ 
          message: "Badge not found" 
        });
      }
      
      res.json(badge);
    } catch (error) {
      console.error("Error fetching badge:", error);
      res.status(500).json({ 
        message: "Failed to fetch badge" 
      });
    }
  });

  app.get("/api/users/:userId/badges", async (req: Request, res: Response) => {
    try {
      const userId = parseInt(req.params.userId);
      const badges = await storage.getBadgesByUser(userId);
      res.json(badges);
    } catch (error) {
      console.error("Error fetching badges for user:", error);
      res.status(500).json({ 
        message: "Failed to fetch badges for user" 
      });
    }
  });

  app.get("/api/dois/:doiId/badges", async (req: Request, res: Response) => {
    try {
      const doiId = parseInt(req.params.doiId);
      const badges = await storage.getBadgesByDoi(doiId);
      res.json(badges);
    } catch (error) {
      console.error("Error fetching badges for DOI:", error);
      res.status(500).json({ 
        message: "Failed to fetch badges for DOI" 
      });
    }
  });

  app.post("/api/badges", async (req: Request, res: Response) => {
    try {
      const user = getAuthUser(req);
      if (!user) {
        return res.status(401).json({ message: "Authentication required" });
      }
      
      const validatedData = insertBadgeSchema.parse(req.body);
      const badge = await storage.createBadge(validatedData);
      
      res.status(201).json(badge);
    } catch (error) {
      console.error("Error creating badge:", error);
      if (error instanceof z.ZodError) {
        return res.status(400).json({ 
          message: "Validation error", 
          errors: error.errors 
        });
      }
      res.status(500).json({ 
        message: "Failed to create badge" 
      });
    }
  });

  // Wallet connection endpoint
  app.post("/api/wallet/connect", (req: Request, res: Response) => {
    try {
      const { walletAddress } = req.body;
      
      if (!walletAddress) {
        return res.status(400).json({ 
          message: "Wallet address is required" 
        });
      }
      
      // Here we would validate the wallet signature, but for this demo just return success
      res.json({ 
        success: true, 
        walletAddress 
      });
    } catch (error) {
      console.error("Error connecting wallet:", error);
      res.status(500).json({ 
        message: "Failed to connect wallet" 
      });
    }
  });

  // Initialize HTTP server
  const httpServer = createServer(app);
  
  // Initialize WebSocket server on a different path
  const wss = new WebSocketServer({ server: httpServer, path: '/ws' });
  
  wss.on('connection', (ws) => {
    console.log('WebSocket client connected');
    
    ws.on('message', (message) => {
      try {
        const data = JSON.parse(message.toString());
        console.log('Received:', data);
        
        // Echo the message back to the client
        if (ws.readyState === ws.OPEN) {
          ws.send(JSON.stringify({
            type: 'response',
            message: 'Received your message',
            data
          }));
        }
      } catch (error) {
        console.error('Error processing WebSocket message:', error);
      }
    });
    
    ws.on('close', () => {
      console.log('WebSocket client disconnected');
    });
    
    // Send initial connection message
    ws.send(JSON.stringify({
      type: 'info',
      message: 'Connected to Fronsciers WebSocket server'
    }));
  });

  return httpServer;
}
```

## Storage Layer
File: `server/storage.ts`

```typescript
import { db } from "./db";
import { eq, desc, and, sql, count, gt, between } from "drizzle-orm";
import {
  users, dois, researcherDoci, badges, transactions, doiResolutions,
  submissions, reviewers, reviews, citations,
  type User, type InsertUser,
  type Doi, type InsertDoi,
  type ResearcherDoci, type InsertResearcherDoci,
  type Badge, type InsertBadge, type BadgeType,
  type Transaction, type InsertTransaction,
  type DoiResolution, type InsertDoiResolution,
  type Submission, type InsertSubmission,
  type Reviewer, type InsertReviewer,
  type Review, type InsertReview,
  type Citation, type InsertCitation
} from "@shared/schema";

export interface IStorage {
  // User operations
  getUser(id: number): Promise<User | undefined>;
  getUserByUsername(username: string): Promise<User | undefined>;
  getUserByWalletAddress(walletAddress: string): Promise<User | undefined>;
  createUser(user: InsertUser): Promise<User>;
  
  // Researcher DOCI operations
  getResearcherDoci(id: number): Promise<ResearcherDoci | undefined>;
  getResearcherDociByUserId(userId: number): Promise<ResearcherDoci | undefined>;
  getResearcherDociByFullDoci(fullDoci: string): Promise<ResearcherDoci | undefined>;
  createResearcherDoci(doci: InsertResearcherDoci): Promise<ResearcherDoci>;
  updateResearcherDoci(id: number, doci: Partial<InsertResearcherDoci>): Promise<ResearcherDoci | undefined>;
  getNextResearcherDoiSuffix(): Promise<string>;
  
  // DOI operations
  getDoi(id: number): Promise<Doi | undefined>;
  getDoiByFullDoi(fullDoi: string): Promise<Doi | undefined>;
  getDoisByUser(userId: number): Promise<Doi[]>;
  getRecentDois(limit?: number): Promise<Doi[]>;
  createDoi(doi: InsertDoi): Promise<Doi>;
  updateDoi(id: number, doi: Partial<InsertDoi>): Promise<Doi | undefined>;
  getDoiStats(): Promise<{ totalDois: number, resolutionCount: number, transactionCount: number }>;
  getNextDoiSuffix(): Promise<string>;
  
  // Transaction operations
  getTransaction(id: number): Promise<Transaction | undefined>;
  getTransactionsByUser(userId: number): Promise<Transaction[]>;
  getTransactionsByDoi(doiId: number): Promise<Transaction[]>;
  getRecentTransactions(limit?: number): Promise<Transaction[]>;
  createTransaction(tx: InsertTransaction): Promise<Transaction>;
  
  // Resolution operations
  recordDoiResolution(resolution: InsertDoiResolution): Promise<DoiResolution>;
  getDoiResolutionCountByDoi(doiId: number): Promise<number>;
  
  // Badge operations (SPL Tokens)
  getBadge(id: number): Promise<Badge | undefined>;
  getBadgesByUser(userId: number): Promise<Badge[]>;
  getBadgesByDoi(doiId: number): Promise<Badge[]>;
  getBadgesByType(badgeType: BadgeType): Promise<Badge[]>;
  createBadge(badge: InsertBadge): Promise<Badge>;
  getRecentBadges(limit?: number): Promise<Badge[]>;
  getBadgeStats(): Promise<{ totalBadges: number, badgesByType: { type: string, count: number }[] }>;
  
  // Submission operations
  getSubmission(id: number): Promise<Submission | undefined>;
  getSubmissionById(submissionId: string): Promise<Submission | undefined>;
  getSubmissionsByUser(authorId: string): Promise<Submission[]>;
  getRecentSubmissions(limit?: number): Promise<Submission[]>;
  createSubmission(submission: InsertSubmission): Promise<Submission>;
  updateSubmission(id: number, submission: Partial<InsertSubmission>): Promise<Submission | undefined>;
  
  // Reviewer operations
  getReviewer(id: number): Promise<Reviewer | undefined>;
  getReviewerByWallet(reviewerId: string): Promise<Reviewer | undefined>;
  createReviewer(reviewer: InsertReviewer): Promise<Reviewer>;
  updateReviewer(id: number, reviewer: Partial<InsertReviewer>): Promise<Reviewer | undefined>;
  
  // Review operations
  getReview(id: number): Promise<Review | undefined>;
  getReviewById(reviewId: string): Promise<Review | undefined>;
  getReviewsByReviewer(reviewerId: string): Promise<Review[]>;
  getReviewsBySubmission(submissionId: string): Promise<Review[]>;
  createReview(review: InsertReview): Promise<Review>;
  updateReview(id: number, review: Partial<InsertReview>): Promise<Review | undefined>;
  
  // Citation operations
  getCitation(id: number): Promise<Citation | undefined>;
  getCitationsByCitingDoi(citingDoiId: string): Promise<Citation[]>;
  getCitationsByCitedDoi(citedDoiId: string): Promise<Citation[]>;
  createCitation(citation: InsertCitation): Promise<Citation>;
  
  // Analytics operations
  getDoiResolutionsByTimeframe(timeframe: 'day' | 'week' | 'month' | 'year'): Promise<{ date: string, count: number }[]>;
  getDoisByType(): Promise<{ type: string, count: number }[]>;
  getTopDoisByResolution(limit?: number): Promise<{ doiId: number, fullDoi: string, title: string, count: number }[]>;
  getCitationStats(): Promise<{ totalCitations: number, avgCitationsPerDoi: number }>;
  
  // Subscription management
  updateStripeCustomerId(userId: number, customerId: string): Promise<User>;
  updateUserStripeInfo(userId: number, stripeInfo: { customerId: string, subscriptionId: string }): Promise<User>;
}

export class DatabaseStorage implements IStorage {
  // Implementation for all methods defined in the IStorage interface
  // Example implementation of key methods:
  
  async getUser(id: number): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.id, id));
    return user || undefined;
  }
  
  async getUserByUsername(username: string): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.username, username));
    return user || undefined;
  }
  
  async getUserByWalletAddress(walletAddress: string): Promise<User | undefined> {
    const [user] = await db.select().from(users).where(eq(users.walletAddress, walletAddress));
    return user || undefined;
  }
  
  async createUser(insertUser: InsertUser): Promise<User> {
    const [user] = await db.insert(users).values(insertUser).returning();
    return user;
  }
  
  async getDoi(id: number): Promise<Doi | undefined> {
    const [doi] = await db.select().from(dois).where(eq(dois.id, id));
    return doi || undefined;
  }
  
  async getDoiByFullDoi(fullDoi: string): Promise<Doi | undefined> {
    const [prefix, suffix] = fullDoi.split('/');
    const [doi] = await db
      .select()
      .from(dois)
      .where(and(
        eq(dois.prefix, prefix),
        eq(dois.suffix, suffix)
      ));
    return doi || undefined;
  }
  
  async createDoi(insertDoi: InsertDoi): Promise<Doi> {
    const [doi] = await db.insert(dois).values(insertDoi).returning();
    return doi;
  }
  
  async recordDoiResolution(insertResolution: InsertDoiResolution): Promise<DoiResolution> {
    const [resolution] = await db
      .insert(doiResolutions)
      .values(insertResolution)
      .returning();
    return resolution;
  }
  
  async getResearcherDociByFullDoci(fullDoci: string): Promise<ResearcherDoci | undefined> {
    const [prefix, suffix] = fullDoci.split('/');
    const [doci] = await db
      .select()
      .from(researcherDoci)
      .where(and(
        eq(researcherDoci.prefix, prefix),
        eq(researcherDoci.suffix, suffix)
      ));
    return doci || undefined;
  }
  
  // Additional methods for all entities (researchers, submissions, reviews, etc.)
  // are implemented similarly using Drizzle ORM methods
  
  // Subscription management methods
  async updateStripeCustomerId(userId: number, customerId: string): Promise<User> {
    const [user] = await db
      .update(users)
      .set({ stripeCustomerId: customerId })
      .where(eq(users.id, userId))
      .returning();
    return user;
  }
  
  async updateUserStripeInfo(userId: number, stripeInfo: { customerId: string, subscriptionId: string }): Promise<User> {
    const [user] = await db
      .update(users)
      .set({
        stripeCustomerId: stripeInfo.customerId,
        stripeSubscriptionId: stripeInfo.subscriptionId,
        subscriptionStatus: 'active'
      })
      .where(eq(users.id, userId))
      .returning();
    return user;
  }
}

// Export a singleton instance
export const storage = new DatabaseStorage();
```

## Solana Smart Contracts

### APC Escrow Program
File: `solana-program/apc-escrow/src/lib.rs`

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{self, Mint, Token, TokenAccount, Transfer};
use anchor_spl::associated_token::AssociatedToken;

declare_id!("ApcESCxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx");

#[program]
pub mod apc_escrow {
    use super::*;

    // Initialize an escrow for an Article Processing Charge
    pub fn initialize(
        ctx: Context<Initialize>,
        manuscript_id: u64,
        amount: u64,
        required_approvals: u8,
    ) -> Result<()> {
        let escrow = &mut ctx.accounts.escrow;
        escrow.payer = ctx.accounts.payer.key();
        escrow.manuscript_id = manuscript_id;
        escrow.amount = amount;
        escrow.required_approvals = required_approvals;
        escrow.approval_count = 0;
        escrow.is_funded = false;
        escrow.is_refunded = false;
        escrow.is_released = false;
        escrow.token_mint = ctx.accounts.token_mint.key();
        escrow.escrow_token_account = ctx.accounts.escrow_token_account.key();
        escrow.approvers = vec![];
        escrow.bump = *ctx.bumps.get("escrow").unwrap();
        Ok(())
    }

    // Fund the escrow with tokens
    pub fn fund(ctx: Context<Fund>, amount: u64) -> Result<()> {
        let escrow = &mut ctx.accounts.escrow;
        require!(!escrow.is_funded, EscrowError::AlreadyFunded);
        require!(amount == escrow.amount, EscrowError::InvalidAmount);

        // Transfer tokens from payer to escrow token account
        let transfer_ix = Transfer {
            from: ctx.accounts.payer_token_account.to_account_info(),
            to: ctx.accounts.escrow_token_account.to_account_info(),
            authority: ctx.accounts.payer.to_account_info(),
        };

        let cpi_ctx = CpiContext::new(
            ctx.accounts.token_program.to_account_info(),
            transfer_ix,
        );

        token::transfer(cpi_ctx, amount)?;
        escrow.is_funded = true;
        Ok(())
    }

    // Approve the manuscript for payment release
    pub fn approve(ctx: Context<Approve>) -> Result<()> {
        let escrow = &mut ctx.accounts.escrow;
        require!(escrow.is_funded, EscrowError::NotFunded);
        require!(!escrow.is_released, EscrowError::AlreadyReleased);
        require!(!escrow.is_refunded, EscrowError::AlreadyRefunded);

        // Check if the reviewer has already approved
        let reviewer_key = ctx.accounts.approver.key();
        require!(
            !escrow.approvers.contains(&reviewer_key),
            EscrowError::AlreadyApproved
        );

        // Add the reviewer to the list of approvers
        escrow.approvers.push(reviewer_key);
        escrow.approval_count += 1;

        Ok(())
    }

    // Release funds to the publisher when required approvals are met
    pub fn release(ctx: Context<Release>) -> Result<()> {
        let escrow = &mut ctx.accounts.escrow;
        require!(escrow.is_funded, EscrowError::NotFunded);
        require!(!escrow.is_released, EscrowError::AlreadyReleased);
        require!(!escrow.is_refunded, EscrowError::AlreadyRefunded);
        require!(
            escrow.approval_count >= escrow.required_approvals,
            EscrowError::InsufficientApprovals
        );

        // Get the escrow PDA signer seeds
        let escrow_seeds = &[
            b"apc_escrow",
            &escrow.manuscript_id.to_le_bytes(),
            &escrow.payer.to_bytes(),
            &[escrow.bump],
        ];
        let signer = &[&escrow_seeds[..]];

        // Transfer tokens from escrow to recipient
        let transfer_ix = Transfer {
            from: ctx.accounts.escrow_token_account.to_account_info(),
            to: ctx.accounts.recipient_token_account.to_account_info(),
            authority: ctx.accounts.escrow.to_account_info(),
        };

        let cpi_ctx = CpiContext::new_with_signer(
            ctx.accounts.token_program.to_account_info(),
            transfer_ix,
            signer,
        );

        token::transfer(cpi_ctx, escrow.amount)?;
        escrow.is_released = true;
        Ok(())
    }

    // Refund the payer if manuscript is rejected
    pub fn refund(ctx: Context<Refund>) -> Result<()> {
        let escrow = &mut ctx.accounts.escrow;
        require!(escrow.is_funded, EscrowError::NotFunded);
        require!(!escrow.is_released, EscrowError::AlreadyReleased);
        require!(!escrow.is_refunded, EscrowError::AlreadyRefunded);

        // Get the escrow PDA signer seeds
        let escrow_seeds = &[
            b"apc_escrow",
            &escrow.manuscript_id.to_le_bytes(),
            &escrow.payer.to_bytes(),
            &[escrow.bump],
        ];
        let signer = &[&escrow_seeds[..]];

        // Transfer tokens from escrow back to payer
        let transfer_ix = Transfer {
            from: ctx.accounts.escrow_token_account.to_account_info(),
            to: ctx.accounts.payer_token_account.to_account_info(),
            authority: ctx.accounts.escrow.to_account_info(),
        };

        let cpi_ctx = CpiContext::new_with_signer(
            ctx.accounts.token_program.to_account_info(),
            transfer_ix,
            signer,
        );

        token::transfer(cpi_ctx, escrow.amount)?;
        escrow.is_refunded = true;
        Ok(())
    }
}

// Account structs for the program

#[account]
pub struct Escrow {
    pub payer: Pubkey,             // The author who paid for APC
    pub manuscript_id: u64,        // Unique ID of the manuscript
    pub amount: u64,               // Amount in token's smallest units
    pub required_approvals: u8,    // Number of approvals required for release
    pub approval_count: u8,        // Current number of approvals
    pub is_funded: bool,           // Whether escrow has been funded
    pub is_released: bool,         // Whether funds have been released
    pub is_refunded: bool,         // Whether funds have been refunded
    pub token_mint: Pubkey,        // Token mint address (e.g., USDC)
    pub escrow_token_account: Pubkey, // Token account holding the funds
    pub approvers: Vec<Pubkey>,    // List of reviewers who have approved
    pub bump: u8,                  // PDA bump seed
}

// Context structs for instructions

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(mut)]
    pub payer: Signer<'info>,
    
    #[account(
        init,
        payer = payer,
        space = 8 + std::mem::size_of::<Escrow>() + 32 * 10, // Space for approvers
        seeds = [
            b"apc_escrow",
            manuscript_id.to_le_bytes().as_ref(),
            payer.key().as_ref()
        ],
        bump
    )]
    pub escrow: Account<'info, Escrow>,
    
    pub token_mint: Account<'info, Mint>,
    
    #[account(
        init,
        payer = payer,
        associated_token::mint = token_mint,
        associated_token::authority = escrow
    )]
    pub escrow_token_account: Account<'info, TokenAccount>,
    
    pub system_program: Program<'info, System>,
    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub rent: Sysvar<'info, Rent>,
}

#[derive(Accounts)]
pub struct Fund<'info> {
    #[account(mut)]
    pub payer: Signer<'info>,
    
    #[account(
        mut,
        seeds = [
            b"apc_escrow",
            escrow.manuscript_id.to_le_bytes().as_ref(),
            escrow.payer.as_ref()
        ],
        bump = escrow.bump,
        constraint = escrow.payer == payer.key()
    )]
    pub escrow: Account<'info, Escrow>,
    
    #[account(
        mut,
        constraint = payer_token_account.owner == payer.key(),
        constraint = payer_token_account.mint == escrow.token_mint
    )]
    pub payer_token_account: Account<'info, TokenAccount>,
    
    #[account(
        mut,
        constraint = escrow_token_account.key() == escrow.escrow_token_account,
        constraint = escrow_token_account.mint == escrow.token_mint
    )]
    pub escrow_token_account: Account<'info, TokenAccount>,
    
    pub token_program: Program<'info, Token>,
}

#[derive(Accounts)]
pub struct Approve<'info> {
    #[account(mut)]
    pub approver: Signer<'info>,
    
    #[account(
        mut,
        seeds = [
            b"apc_escrow",
            escrow.manuscript_id.to_le_bytes().as_ref(),
            escrow.payer.as_ref()
        ],
        bump = escrow.bump
    )]
    pub escrow: Account<'info, Escrow>,
}

#[derive(Accounts)]
pub struct Release<'info> {
    #[account(mut)]
    pub publisher: Signer<'info>,
    
    #[account(
        mut,
        seeds = [
            b"apc_escrow",
            escrow.manuscript_id.to_le_bytes().as_ref(),
            escrow.payer.as_ref()
        ],
        bump = escrow.bump
    )]
    pub escrow: Account<'info, Escrow>,
    
    #[account(
        mut,
        constraint = escrow_token_account.key() == escrow.escrow_token_account,
        constraint = escrow_token_account.mint == escrow.token_mint
    )]
    pub escrow_token_account: Account<'info, TokenAccount>,
    
    #[account(
        mut,
        constraint = recipient_token_account.owner == publisher.key(),
        constraint = recipient_token_account.mint == escrow.token_mint
    )]
    pub recipient_token_account: Account<'info, TokenAccount>,
    
    pub token_program: Program<'info, Token>,
}

#[derive(Accounts)]
pub struct Refund<'info> {
    pub refund_authority: Signer<'info>,
    
    #[account(
        mut,
        seeds = [
            b"apc_escrow",
            escrow.manuscript_id.to_le_bytes().as_ref(),
            escrow.payer.as_ref()
        ],
        bump = escrow.bump
    )]
    pub escrow: Account<'info, Escrow>,
    
    #[account(
        mut,
        constraint = escrow_token_account.key() == escrow.escrow_token_account,
        constraint = escrow_token_account.mint == escrow.token_mint
    )]
    pub escrow_token_account: Account<'info, TokenAccount>,
    
    #[account(
        mut,
        constraint = payer_token_account.owner == escrow.payer,
        constraint = payer_token_account.mint == escrow.token_mint
    )]
    pub payer_token_account: Account<'info, TokenAccount>,
    
    pub token_program: Program<'info, Token>,
}

// Error codes for the program
#[error_code]
pub enum EscrowError {
    #[msg("Escrow is already funded")]
    AlreadyFunded,
    #[msg("Escrow is not yet funded")]
    NotFunded,
    #[msg("Funds have already been released")]
    AlreadyReleased,
    #[msg("Funds have already been refunded")]
    AlreadyRefunded,
    #[msg("Amount must match the escrow amount")]
    InvalidAmount,
    #[msg("This reviewer has already approved")]
    AlreadyApproved,
    #[msg("Not enough approvals to release funds")]
    InsufficientApprovals,
}
```

### Badge Program
File: `solana-program/badge-program/src/lib.rs`

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{self, Mint, Token, TokenAccount};
use anchor_spl::associated_token::AssociatedToken;

declare_id!("BadgeXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX");

#[program]
pub mod badge_program {
    use super::*;

    // Create a new badge type (admin only)
    pub fn create_badge_type(
        ctx: Context<CreateBadgeType>,
        badge_type: BadgeType,
        name: String,
        description: String,
        supply_cap: Option<u64>,
    ) -> Result<()> {
        let badge_type_account = &mut ctx.accounts.badge_type_account;
        badge_type_account.authority = ctx.accounts.authority.key();
        badge_type_account.badge_type = badge_type as u8;
        badge_type_account.name = name;
        badge_type_account.description = description;
        badge_type_account.supply_cap = supply_cap;
        badge_type_account.current_supply = 0;
        badge_type_account.token_mint = ctx.accounts.token_mint.key();
        badge_type_account.is_transferable = false; // Academic badges are typically non-transferable
        badge_type_account.bump = *ctx.bumps.get("badge_type_account").unwrap();
        
        Ok(())
    }

    // Issue a badge to a reviewer
    pub fn issue_badge(
        ctx: Context<IssueBadge>,
        badge_type: BadgeType,
        recipient_id: String,
        metadata_uri: String,
    ) -> Result<()> {
        let badge_type_account = &mut ctx.accounts.badge_type_account;
        let badge_account = &mut ctx.accounts.badge_account;
        
        // Check supply cap if it exists
        if let Some(cap) = badge_type_account.supply_cap {
            require!(
                badge_type_account.current_supply < cap,
                BadgeError::SupplyCapExceeded
            );
        }
        
        // Set up badge account data
        badge_account.badge_type = badge_type as u8;
        badge_account.recipient = ctx.accounts.recipient.key();
        badge_account.issuer = ctx.accounts.issuer.key();
        badge_account.issue_date = Clock::get()?.unix_timestamp;
        badge_account.token_mint = badge_type_account.token_mint;
        badge_account.recipient_token_account = ctx.accounts.recipient_token_account.key();
        badge_account.recipient_id = recipient_id;
        badge_account.metadata_uri = metadata_uri;
        badge_account.revoked = false;
        badge_account.bump = *ctx.bumps.get("badge_account").unwrap();

        // Mint 1 token to the recipient's token account
        let seeds = &[
            b"badge_type",
            &[badge_type as u8],
            &badge_type_account.authority.to_bytes(),
            &[badge_type_account.bump],
        ];
        let signer = &[&seeds[..]];
        
        let cpi_accounts = token::MintTo {
            mint: ctx.accounts.token_mint.to_account_info(),
            to: ctx.accounts.recipient_token_account.to_account_info(),
            authority: ctx.accounts.badge_type_account.to_account_info(),
        };
        
        let cpi_program = ctx.accounts.token_program.to_account_info();
        let cpi_ctx = CpiContext::new_with_signer(cpi_program, cpi_accounts, signer);
        
        token::mint_to(cpi_ctx, 1)?;
        
        // Increment badge type supply
        badge_type_account.current_supply += 1;
        
        Ok(())
    }
    
    // Revoke a badge
    pub fn revoke_badge(ctx: Context<RevokeBadge>) -> Result<()> {
        let badge_account = &mut ctx.accounts.badge_account;
        require!(!badge_account.revoked, BadgeError::AlreadyRevoked);
        
        badge_account.revoked = true;
        
        // Burn the token
        let seeds = &[
            b"badge",
            &badge_account.recipient.to_bytes(),
            &[badge_account.badge_type],
            &[badge_account.bump],
        ];
        let signer = &[&seeds[..]];
        
        let cpi_accounts = token::Burn {
            mint: ctx.accounts.token_mint.to_account_info(),
            from: ctx.accounts.recipient_token_account.to_account_info(),
            authority: ctx.accounts.badge_account.to_account_info(),
        };
        
        let cpi_program = ctx.accounts.token_program.to_account_info();
        let cpi_ctx = CpiContext::new_with_signer(cpi_program, cpi_accounts, signer);
        
        token::burn(cpi_ctx, 1)?;
        
        // Decrement badge type supply
        let badge_type_account = &mut ctx.accounts.badge_type_account;
        badge_type_account.current_supply -= 1;
        
        Ok(())
    }
}

// Badge types for reviewers
#[derive(Clone, Copy, Debug, PartialEq, AnchorSerialize, AnchorDeserialize)]
pub enum BadgeType {
    Bronze = 0,
    Silver = 1,
    Gold = 2,
    Platinum = 3,
}

// Account structs
#[account]
pub struct BadgeTypeAccount {
    pub authority: Pubkey,       // Admin authority for this badge type
    pub badge_type: u8,          // Badge type identifier
    pub name: String,            // Badge name
    pub description: String,     // Badge description
    pub supply_cap: Option<u64>, // Maximum supply (optional)
    pub current_supply: u64,     // Current number of badges issued
    pub token_mint: Pubkey,      // SPL token mint address
    pub is_transferable: bool,   // Whether badge tokens can be transferred
    pub bump: u8,                // PDA bump seed
}

#[account]
pub struct BadgeAccount {
    pub badge_type: u8,                 // Badge type identifier
    pub recipient: Pubkey,              // Recipient wallet address
    pub issuer: Pubkey,                 // Issuer wallet address
    pub issue_date: i64,                // Unix timestamp of issuance
    pub token_mint: Pubkey,             // SPL token mint address
    pub recipient_token_account: Pubkey, // Recipient's token account
    pub recipient_id: String,           // Reviewer's identifier in system
    pub metadata_uri: String,           // URI to badge metadata
    pub revoked: bool,                  // Whether badge has been revoked
    pub bump: u8,                       // PDA bump seed
}

// Context structs for instructions
#[derive(Accounts)]
pub struct CreateBadgeType<'info> {
    #[account(mut)]
    pub authority: Signer<'info>,
    
    #[account(
        init,
        payer = authority,
        space = 8 + std::mem::size_of::<BadgeTypeAccount>() + 32 + 256, // Extra space for strings
        seeds = [
            b"badge_type",
            &[badge_type as u8],
            authority.key().as_ref()
        ],
        bump
    )]
    pub badge_type_account: Account<'info, BadgeTypeAccount>,
    
    // Create a new token mint for this badge type
    #[account(
        init,
        payer = authority,
        mint::decimals = 0, // No decimals for badges
        mint::authority = badge_type_account
    )]
    pub token_mint: Account<'info, Mint>,
    
    pub system_program: Program<'info, System>,
    pub token_program: Program<'info, Token>,
    pub rent: Sysvar<'info, Rent>,
}

#[derive(Accounts)]
#[instruction(badge_type: BadgeType, recipient_id: String)]
pub struct IssueBadge<'info> {
    #[account(mut)]
    pub issuer: Signer<'info>,
    
    pub recipient: AccountInfo<'info>,
    
    #[account(
        mut,
        seeds = [
            b"badge_type",
            &[badge_type as u8],
            badge_type_account.authority.as_ref()
        ],
        bump = badge_type_account.bump
    )]
    pub badge_type_account: Account<'info, BadgeTypeAccount>,
    
    #[account(
        init,
        payer = issuer,
        space = 8 + std::mem::size_of::<BadgeAccount>() + 256, // Extra space for strings
        seeds = [
            b"badge",
            recipient.key().as_ref(),
            &[badge_type as u8],
            issuer.key().as_ref()
        ],
        bump
    )]
    pub badge_account: Account<'info, BadgeAccount>,
    
    #[account(
        mut,
        constraint = token_mint.key() == badge_type_account.token_mint
    )]
    pub token_mint: Account<'info, Mint>,
    
    #[account(
        init_if_needed,
        payer = issuer,
        associated_token::mint = token_mint,
        associated_token::authority = recipient
    )]
    pub recipient_token_account: Account<'info, TokenAccount>,
    
    pub system_program: Program<'info, System>,
    pub token_program: Program<'info, Token>,
    pub associated_token_program: Program<'info, AssociatedToken>,
    pub rent: Sysvar<'info, Rent>,
}

#[derive(Accounts)]
pub struct RevokeBadge<'info> {
    #[account(mut)]
    pub authority: Signer<'info>,
    
    #[account(
        mut,
        constraint = badge_account.issuer == authority.key() || badge_type_account.authority == authority.key()
    )]
    pub badge_account: Account<'info, BadgeAccount>,
    
    #[account(
        mut,
        seeds = [
            b"badge_type",
            &[badge_account.badge_type],
            badge_type_account.authority.as_ref()
        ],
        bump = badge_type_account.bump
    )]
    pub badge_type_account: Account<'info, BadgeTypeAccount>,
    
    #[account(
        mut,
        constraint = token_mint.key() == badge_account.token_mint
    )]
    pub token_mint: Account<'info, Mint>,
    
    #[account(
        mut,
        constraint = recipient_token_account.key() == badge_account.recipient_token_account
    )]
    pub recipient_token_account: Account<'info, TokenAccount>,
    
    pub token_program: Program<'info, Token>,
}

#[error_code]
pub enum BadgeError {
    #[msg("Badge supply cap has been exceeded")]
    SupplyCapExceeded,
    #[msg("This badge has already been revoked")]
    AlreadyRevoked,
}
```

## Frontend Components

### ResearcherProfile Component
File: `client/src/pages/ResearcherProfile.tsx`

```tsx
import React, { useState, useEffect } from "react";
import { useParams, Link } from "wouter";
import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import {
  Tabs,
  TabsContent,
  TabsList,
  TabsTrigger,
} from "@/components/ui/tabs";
import { Button } from "@/components/ui/button";
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar";
import { Badge } from "@/components/ui/badge";
import {
  MapPinIcon,
  PhoneIcon,
  MailIcon,
  LinkIcon,
  BriefcaseIcon,
  GraduationCapIcon,
  BookOpenIcon,
  DownloadIcon,
  CopyIcon,
  ClipboardIcon,
  AwardIcon,
  CalendarIcon,
  ExternalLinkIcon,
} from "lucide-react";
import { useToast } from "@/hooks/use-toast";
import { apiRequest } from "@/lib/queryClient";

const ResearcherProfile = () => {
  const { dociId } = useParams<{ dociId: string }>();
  const { toast } = useToast();
  const [researcher, setResearcher] = useState<any>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [activeTab, setActiveTab] = useState<string>("about");

  useEffect(() => {
    const fetchResearcher = async () => {
      try {
        const response = await apiRequest("GET", `/api/researchers/${dociId}`);
        const data = await response.json();
        setResearcher(data);
      } catch (error) {
        console.error("Error fetching researcher:", error);
        
        const fallbackResearcher = {
          id: 1,
          doci: "10.FRONS-R/R-7A8B9C",
          name: "Rejoel Mangasa Siagian",
          email: "rejoel.mangasa@ui.ac.id",
          phone: "+6282246486128",
          location: "Salemba, Jakarta Pusat",
          profileImage: "https://api.dicebear.com/7.x/initials/svg?seed=RS&background=%230496FF",
          bio: "Rejoel Mangasa Siagian is a medical doctor from the Faculty of Medicine, Universitas Indonesia, having extensively studied evidence-based medicine, community empowerment, and the integration of medical technology. With a keen interest in primary prevention and clinical nutrition, he aims to make meaningful contributions to healthcare equity. His practical experiences in scientific writing and medical research fuel his ambition to become a clinician and researcher. Let's discuss and bring noteworthy initiatives in healthcare advancement to clinical practices.",
          socialLinks: [
            {
              platform: "LinkedIn",
              url: "https://www.linkedin.com/in/rejoelmangasa/",
            }
          ],
          education: [
            {
              institution: "Universitas Indonesia",
              degree: "Doctor of Medicine",
              fieldOfStudy: "Medical Sciences",
              startDate: "2018",
              endDate: "2024",
              description: [
                "Specialized in Clinical Nutrition and Primary Prevention",
                "Completed evidence-based medicine research and community health projects",
                "Led multiple healthcare technology integration initiatives",
                "Graduated with honors"
              ]
            },
            {
              institution: "Jakarta International College",
              degree: "Foundation Year",
              fieldOfStudy: "Pre-Medicine",
              startDate: "2017",
              endDate: "2018",
              description: [
                "Completed preparatory coursework in Biology, Chemistry, and Physics",
                "Received scholarship for academic excellence"
              ]
            }
          ],
          experience: [
            {
              position: "Research Assistant",
              organization: "University of Indonesia Medical Research Center",
              type: "Part-time",
              location: "Jakarta, Indonesia",
              startDate: "Jan 2022",
              endDate: "Present",
              description: [
                "Assisted in clinical trials for nutrition-based interventions in type 2 diabetes",
                "Conducted literature reviews and meta-analyses for evidence-based medicine projects",
                "Developed data collection tools for community health surveys",
                "Contributed to 3 peer-reviewed publications"
              ]
            },
            {
              position: "Medical Volunteer",
              organization: "Indonesian Medical Association",
              type: "Volunteer",
              location: "Various locations, Indonesia",
              startDate: "Jun 2021",
              endDate: "Dec 2021",
              description: [
                "Provided basic healthcare services in underserved rural communities",
                "Participated in health education programs focusing on preventive care",
                "Assisted in COVID-19 vaccination campaigns",
                "Collaborated with interdisciplinary teams to address community health needs"
              ]
            }
          ],
          publications: [
            {
              id: 1,
              title: "Efficacy of Plant-Based Diets in Reducing Cardiovascular Risk Factors: A Systematic Review",
              journal: "Indonesian Journal of Clinical Nutrition",
              year: "2023",
              indexing: "Scopus",
              doi: "10.FRONS/67890",
              url: "#"
            },
            {
              id: 2,
              title: "Implementation of Mobile Health Technologies in Rural Indonesia: Challenges and Opportunities",
              journal: "Digital Health",
              year: "2022",
              indexing: "DOAJ",
              doi: "10.FRONS/12345",
              url: "#"
            },
            {
              id: 3,
              title: "Community-Based Approach to Type 2 Diabetes Prevention: A Pilot Study in Jakarta",
              journal: "Southeast Asian Journal of Public Health",
              year: "2021",
              doi: "10.FRONS/54321",
              url: "#"
            }
          ],
          projects: [
            {
              title: "NutriTech Mobile Application Development",
              date: "2023",
              description: "Led the development of a mobile application to provide personalized nutrition guidance based on individual health profiles. Collaborated with software engineers and nutritionists to create an evidence-based recommendation algorithm."
            },
            {
              title: "Rural Health Access Improvement Initiative",
              date: "2022",
              description: "Developed and implemented a program to improve healthcare access in rural communities through telemedicine and health worker training. Resulted in 40% increase in primary care utilization among target population."
            },
            {
              title: "Medical Student Research Conference",
              date: "2021",
              description: "Organized and chaired the annual medical student research conference, featuring 50+ student presentations and workshops on research methodology and scientific writing."
            }
          ],
          awards: [
            {
              title: "Young Researcher Award",
              issuer: "Indonesian Medical Association",
              date: "2023",
              description: "Recognized for outstanding contribution to clinical nutrition research"
            },
            {
              title: "Community Health Innovation Grant",
              issuer: "Ministry of Health Indonesia",
              date: "2022",
              description: "Awarded funding for the development of community-based preventive health initiatives"
            },
            {
              title: "Academic Excellence Scholarship",
              issuer: "Universitas Indonesia",
              date: "2020-2023",
              description: "Full tuition scholarship based on academic performance"
            }
          ]
        };
        
        setResearcher(fallbackResearcher);
      } finally {
        setLoading(false);
      }
    };
    
    fetchResearcher();
  }, [dociId, toast]);

  const copyDociToClipboard = () => {
    if (researcher?.doci) {
      navigator.clipboard.writeText(researcher.doci);
      toast({
        title: "DOCI copied",
        description: "Direct On-Chain Identifier copied to clipboard",
      });
    }
  };

  if (loading) {
    return (
      <div className="flex justify-center items-center h-screen">
        <div className="animate-spin h-10 w-10 border-4 border-primary border-t-transparent rounded-full"></div>
      </div>
    );
  }

  if (!researcher) {
    return (
      <div className="flex flex-col items-center justify-center h-screen">
        <h1 className="text-2xl font-bold mb-4">Researcher Not Found</h1>
        <p className="text-muted-foreground mb-6">
          The researcher DOCI you're looking for doesn't exist or hasn't been registered.
        </p>
        <Button asChild>
          <Link href="/">Return Home</Link>
        </Button>
      </div>
    );
  }

  return (
    <div className="container mx-auto py-8 px-4">
      {/* Researcher Header */}
      <Card className="mb-8 overflow-hidden">
        <div className="bg-gradient-to-r from-odyssey-lighter to-odyssey-light h-32 md:h-48"></div>
        <div className="px-4 md:px-8 pb-6 relative">
          <div className="flex flex-col md:flex-row md:items-end -mt-12 mb-6 gap-6">
            <Avatar className="h-24 w-24 md:h-32 md:w-32 border-4 border-background">
              <AvatarImage src={researcher.profileImage} alt={researcher.name} />
              <AvatarFallback>
                {researcher.name
                  .split(" ")
                  .map((part: string) => part[0])
                  .join("")}
              </AvatarFallback>
            </Avatar>
            <div className="space-y-2 pt-2 md:pb-2">
              <div className="flex flex-col md:flex-row md:items-center gap-2">
                <h1 className="text-2xl md:text-3xl font-bold">{researcher.name}</h1>
                <Badge className="w-fit">Medical Researcher</Badge>
              </div>
              <div className="text-muted-foreground">{researcher.education[0].institution}</div>
              <div className="flex flex-wrap gap-3 text-sm text-muted-foreground">
                <div className="flex items-center">
                  <MapPinIcon className="h-4 w-4 mr-1" />
                  {researcher.location}
                </div>
                <div className="flex items-center">
                  <PhoneIcon className="h-4 w-4 mr-1" />
                  <a href={`tel:${researcher.phone}`} className="hover:underline">
                    {researcher.phone}
                  </a>
                </div>
                <div className="flex items-center">
                  <MailIcon className="h-4 w-4 mr-1" />
                  <a href={`mailto:${researcher.email}`} className="hover:underline">
                    {researcher.email}
                  </a>
                </div>
                {researcher.socialLinks.map((link: any, index: number) => (
                  <div key={index} className="flex items-center">
                    <LinkIcon className="h-4 w-4 mr-1" />
                    <a
                      href={link.url}
                      target="_blank"
                      rel="noopener noreferrer"
                      className="hover:underline"
                    >
                      {link.platform}
                    </a>
                  </div>
                ))}
              </div>
            </div>
            
            <div className="md:ml-auto flex flex-col md:flex-row items-start md:items-center gap-2 mt-4 md:mt-0">
              <div className="flex items-center bg-muted px-3 py-1 rounded-md cursor-pointer" onClick={copyDociToClipboard}>
                <div className="text-sm mr-2">DOCI: <span className="font-mono">{researcher.doci}</span></div>
                <CopyIcon className="h-4 w-4" />
              </div>
              <Button size="sm">
                <DownloadIcon className="h-4 w-4 mr-2" />
                Export CV
              </Button>
            </div>
          </div>
          
          {/* Stats Cards */}
          <div className="grid grid-cols-2 md:grid-cols-4 gap-4 my-6">
            <Card>
              <CardContent className="p-4 flex flex-col items-center justify-center">
                <div className="text-3xl font-bold text-odyssey">3</div>
                <div className="text-xs text-muted-foreground">Publications</div>
              </CardContent>
            </Card>
            <Card>
              <CardContent className="p-4 flex flex-col items-center justify-center">
                <div className="text-3xl font-bold text-odyssey">12</div>
                <div className="text-xs text-muted-foreground">Citations</div>
              </CardContent>
            </Card>
            <Card>
              <CardContent className="p-4 flex flex-col items-center justify-center">
                <div className="text-3xl font-bold text-odyssey">2</div>
                <div className="text-xs text-muted-foreground">h-index</div>
              </CardContent>
            </Card>
            <Card>
              <CardContent className="p-4 flex flex-col items-center justify-center">
                <div className="text-3xl font-bold text-odyssey">3</div>
                <div className="text-xs text-muted-foreground">Projects</div>
              </CardContent>
            </Card>
          </div>
          
          {/* Tabs */}
          <Tabs value={activeTab} onValueChange={setActiveTab}>
            <TabsList className="w-full md:w-auto">
              <TabsTrigger value="about">About</TabsTrigger>
              <TabsTrigger value="experience">Experience</TabsTrigger>
              <TabsTrigger value="publications">Publications</TabsTrigger>
              <TabsTrigger value="projects">Projects</TabsTrigger>
              <TabsTrigger value="awards">Awards</TabsTrigger>
            </TabsList>
            
            {/* About Tab */}
            <TabsContent value="about" className="mt-6">
              <div className="space-y-6">
                <div>
                  <h2 className="text-xl font-semibold mb-3">Bio</h2>
                  <p className="text-muted-foreground">{researcher.bio}</p>
                </div>
                
                <div>
                  <h2 className="text-xl font-semibold mb-3">Education</h2>
                  <div className="space-y-4">
                    {researcher.education.map((edu: any, index: number) => (
                      <Card key={index}>
                        <CardContent className="p-5">
                          <div className="flex items-start">
                            <GraduationCapIcon className="h-5 w-5 text-odyssey mr-3 mt-1" />
                            <div>
                              <div className="font-semibold text-lg">{edu.institution}</div>
                              <div className="font-medium">{edu.degree}, {edu.fieldOfStudy}</div>
                              <div className="text-sm text-muted-foreground mb-2">{edu.startDate} - {edu.endDate}</div>
                              <ul className="list-disc list-inside space-y-1 text-sm pl-2">
                                {edu.description.map((desc: string, i: number) => (
                                  <li key={i}>{desc}</li>
                                ))}
                              </ul>
                            </div>
                          </div>
                        </CardContent>
                      </Card>
                    ))}
                  </div>
                </div>
                
                <div>
                  <h2 className="text-xl font-semibold mb-3">Skills</h2>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    {researcher.skills && researcher.skills.map((skillGroup: any, index: number) => (
                      <Card key={index}>
                        <CardHeader className="pb-2">
                          <CardTitle className="text-base">{skillGroup.category}</CardTitle>
                        </CardHeader>
                        <CardContent>
                          <div className="flex flex-wrap gap-2">
                            {skillGroup.items.map((skill: string, i: number) => (
                              <Badge key={i} variant="secondary">
                                {skill}
                              </Badge>
                            ))}
                          </div>
                        </CardContent>
                      </Card>
                    ))}
                  </div>
                </div>
              </div>
            </TabsContent>
            
            {/* Experience Tab */}
            <TabsContent value="experience" className="mt-6">
              <h2 className="text-xl font-semibold mb-4">Professional Experience</h2>
              <div className="space-y-6">
                {researcher.experience.map((exp: any, index: number) => (
                  <Card key={index}>
                    <CardContent className="p-5">
                      <div className="flex items-start">
                        <BriefcaseIcon className="h-5 w-5 text-odyssey mr-3 mt-1" />
                        <div className="w-full">
                          <div className="flex flex-col md:flex-row justify-between">
                            <div className="font-semibold text-lg">{exp.position}</div>
                            <Badge variant="outline">{exp.type}</Badge>
                          </div>
                          <div className="flex flex-col md:flex-row md:items-center gap-1 md:gap-3 text-sm">
                            <div className="font-medium">{exp.organization}</div>
                            <div className="hidden md:block text-muted-foreground">•</div>
                            <div className="flex items-center">
                              <MapPinIcon className="h-3 w-3 mr-1" />
                              {exp.location}
                            </div>
                          </div>
                          <div className="text-sm text-muted-foreground mb-3">
                            <CalendarIcon className="h-3 w-3 inline mr-1" />
                            {exp.startDate} - {exp.endDate}
                          </div>
                          
                          <ul className="list-disc list-inside space-y-1 text-sm pl-2">
                            {exp.description.map((desc: string, i: number) => (
                              <li key={i}>{desc}</li>
                            ))}
                          </ul>
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </TabsContent>
            
            {/* Publications Tab */}
            <TabsContent value="publications" className="mt-6">
              <div className="flex items-center justify-between mb-4">
                <h2 className="text-xl font-semibold">Publications ({researcher.publications.length})</h2>
                <Button variant="outline" size="sm">
                  <DownloadIcon className="h-4 w-4 mr-2" />
                  Export Citations
                </Button>
              </div>
              
              <div className="space-y-4">
                {researcher.publications.map((pub: any, index: number) => (
                  <Card key={index}>
                    <CardContent className="p-5">
                      <div className="flex items-start">
                        <BookOpenIcon className="h-5 w-5 text-odyssey mr-3 mt-1" />
                        <div>
                          <div className="font-semibold text-lg">{pub.title}</div>
                          <div className="flex flex-col md:flex-row md:items-center gap-1 md:gap-3 text-sm">
                            <div className="font-medium">{pub.journal}</div>
                            <div className="hidden md:block text-muted-foreground">•</div>
                            <div>{pub.year}</div>
                            {pub.indexing && (
                              <>
                                <div className="hidden md:block text-muted-foreground">•</div>
                                <Badge variant="secondary">{pub.indexing}</Badge>
                              </>
                            )}
                          </div>
                          
                          <div className="flex flex-wrap gap-3 mt-3">
                            {pub.id && (
                              <div className="flex items-center text-sm">
                                <div className="font-medium mr-1">DOCI:</div>
                                <Link href={`/resolve/${pub.doi}`} className="text-odyssey hover:underline flex items-center">
                                  {pub.doi}
                                  <ExternalLinkIcon className="h-3 w-3 ml-1" />
                                </Link>
                              </div>
                            )}
                            {pub.url && (
                              <Button variant="link" size="sm" className="h-6 p-0" asChild>
                                <a href={pub.url} target="_blank" rel="noopener noreferrer">
                                  View Publication
                                  <ExternalLinkIcon className="h-3 w-3 ml-1" />
                                </a>
                              </Button>
                            )}
                          </div>
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </TabsContent>
            
            {/* Projects Tab */}
            <TabsContent value="projects" className="mt-6">
              <h2 className="text-xl font-semibold mb-4">Research Projects & Training</h2>
              <div className="space-y-4">
                {researcher.projects.map((project: any, index: number) => (
                  <Card key={index}>
                    <CardContent className="p-5">
                      <div className="flex items-start">
                        <ClipboardIcon className="h-5 w-5 text-odyssey mr-3 mt-1" />
                        <div>
                          <div className="font-semibold text-lg">{project.title}</div>
                          <div className="text-sm text-muted-foreground mb-2">
                            <CalendarIcon className="h-3 w-3 inline mr-1" />
                            {project.date}
                          </div>
                          <div className="text-sm">{project.description}</div>
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </TabsContent>
            
            {/* Awards Tab */}
            <TabsContent value="awards" className="mt-6">
              <h2 className="text-xl font-semibold mb-4">Awards & Recognition</h2>
              <div className="space-y-4">
                {researcher.awards.map((award: any, index: number) => (
                  <Card key={index}>
                    <CardContent className="p-5">
                      <div className="flex items-start">
                        <AwardIcon className="h-5 w-5 text-odyssey mr-3 mt-1" />
                        <div>
                          <div className="font-semibold text-lg">{award.title}</div>
                          <div className="flex flex-col md:flex-row md:items-center gap-1 md:gap-3 text-sm">
                            <div className="font-medium">{award.issuer}</div>
                            <div className="hidden md:block text-muted-foreground">•</div>
                            <div className="text-muted-foreground">
                              <CalendarIcon className="h-3 w-3 inline mr-1" />
                              {award.date}
                            </div>
                          </div>
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            </TabsContent>
          </Tabs>
        </div>
      </Card>
    </div>
  );
};

export default ResearcherProfile;
```

### Solana Escrow Utility
File: `client/src/utils/solanaEscrow.ts`

```typescript
import { useState } from "react";
import { useWallet } from "@solana/wallet-adapter-react";
import {
  Connection,
  PublicKey,
  SystemProgram,
  Transaction,
  LAMPORTS_PER_SOL,
  Keypair,
} from "@solana/web3.js";
import { Program, AnchorProvider, web3, BN } from "@project-serum/anchor";
import {
  ASSOCIATED_TOKEN_PROGRAM_ID,
  createAssociatedTokenAccountInstruction,
  getAssociatedTokenAddress,
  TOKEN_PROGRAM_ID,
} from "@solana/spl-token";

// USDC mint address (devnet)
const USDC_MINT = new PublicKey("EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v");

// Program ID for APC escrow (placeholder, would be actual deployed program ID)
const ESCROW_PROGRAM_ID = new PublicKey("ApcESCxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx");

// Load the IDL
const escrowIdl = {
  /* APC escrow program IDL would be loaded here */
};

// Function to get program instance
function getProgram(connection: Connection, wallet: any) {
  const provider = new AnchorProvider(
    connection,
    wallet,
    AnchorProvider.defaultOptions()
  );
  return new Program(escrowIdl as any, ESCROW_PROGRAM_ID, provider);
}

// Get the escrow PDA
async function getEscrowPDA(manuscriptId: number, payerPubkey: PublicKey) {
  return await PublicKey.findProgramAddress(
    [
      Buffer.from("apc_escrow"),
      new BN(manuscriptId).toArrayLike(Buffer, "le", 8),
      payerPubkey.toBuffer(),
    ],
    ESCROW_PROGRAM_ID
  );
}

// Get the escrow token account PDA
async function getEscrowTokenAccountPDA(escrowPda: PublicKey) {
  return await getAssociatedTokenAddress(
    USDC_MINT,
    escrowPda,
    true // allowOwnerOffCurve
  );
}

export function useApcEscrow() {
  const { publicKey, signTransaction } = useWallet();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const connection = new Connection("https://api.devnet.solana.com", "confirmed");

  // Initialize a new APC escrow account
  const initializeEscrow = async (manuscriptId: number, amount: number, requiredApprovals: number = 3) => {
    if (!publicKey || !signTransaction) {
      setError("Wallet not connected");
      return null;
    }

    setLoading(true);
    setError(null);

    try {
      const program = getProgram(connection, {
        publicKey,
        signTransaction,
      });
      
      // Convert amount to USDC units (6 decimals)
      const usdcAmount = new BN(amount * 1_000_000);
      
      // Get the escrow PDA
      const [escrowPda, escrowBump] = await getEscrowPDA(manuscriptId, publicKey);
      
      // Get the escrow token account PDA
      const escrowTokenAccountPda = await getEscrowTokenAccountPDA(escrowPda);

      // Initialize the escrow on-chain
      const tx = await program.methods
        .initialize(
          new BN(manuscriptId),
          usdcAmount,
          requiredApprovals
        )
        .accounts({
          escrow: escrowPda,
          escrowTokenAccount: escrowTokenAccountPda,
          tokenMint: USDC_MINT,
          payer: publicKey,
          systemProgram: SystemProgram.programId,
          tokenProgram: TOKEN_PROGRAM_ID,
          associatedTokenProgram: ASSOCIATED_TOKEN_PROGRAM_ID,
          rent: web3.SYSVAR_RENT_PUBKEY,
        })
        .rpc();

      return {
        escrowAddress: escrowPda.toString(),
        tokenAccountAddress: escrowTokenAccountPda.toString(),
        transactionSignature: tx,
      };
    } catch (err: any) {
      setError(`Failed to initialize escrow: ${err.message}`);
      console.error("Error initializing escrow:", err);
      return null;
    } finally {
      setLoading(false);
    }
  };

  // Fund the escrow with USDC
  const fundEscrow = async (manuscriptId: number, amount: number) => {
    if (!publicKey || !signTransaction) {
      setError("Wallet not connected");
      return null;
    }

    setLoading(true);
    setError(null);

    try {
      const program = getProgram(connection, {
        publicKey,
        signTransaction,
      });
      
      // Convert amount to USDC units (6 decimals)
      const usdcAmount = new BN(amount * 1_000_000);
      
      // Get the escrow PDA
      const [escrowPda] = await getEscrowPDA(manuscriptId, publicKey);
      
      // Get the escrow token account PDA
      const escrowTokenAccountPda = await getEscrowTokenAccountPDA(escrowPda);
      
      // Get the payer's USDC token account
      const payerTokenAccount = await getAssociatedTokenAddress(
        USDC_MINT,
        publicKey
      );

      // Fund the escrow
      const tx = await program.methods
        .fund(usdcAmount)
        .accounts({
          escrow: escrowPda,
          escrowTokenAccount: escrowTokenAccountPda,
          payerTokenAccount: payerTokenAccount,
          payer: publicKey,
          tokenProgram: TOKEN_PROGRAM_ID,
        })
        .rpc();

      return {
        transactionSignature: tx,
      };
    } catch (err: any) {
      setError(`Failed to fund escrow: ${err.message}`);
      console.error("Error funding escrow:", err);
      return null;
    } finally {
      setLoading(false);
    }
  };

  // Approve a manuscript for payment release (as a reviewer)
  const approveEscrow = async (manuscriptId: number, payerPublicKey: string) => {
    if (!publicKey || !signTransaction) {
      setError("Wallet not connected");
      return null;
    }

    setLoading(true);
    setError(null);

    try {
      const program = getProgram(connection, {
        publicKey,
        signTransaction,
      });
      
      // Get the escrow PDA using payer's public key
      const [escrowPda] = await getEscrowPDA(
        manuscriptId,
        new PublicKey(payerPublicKey)
      );

      // Approve the escrow
      const tx = await program.methods
        .approve()
        .accounts({
          escrow: escrowPda,
          approver: publicKey,
        })
        .rpc();

      return {
        transactionSignature: tx,
      };
    } catch (err: any) {
      setError(`Failed to approve escrow: ${err.message}`);
      console.error("Error approving escrow:", err);
      return null;
    } finally {
      setLoading(false);
    }
  };

  // Release funds to publisher when required approvals are met
  const releaseEscrow = async (manuscriptId: number, payerPublicKey: string) => {
    if (!publicKey || !signTransaction) {
      setError("Wallet not connected");
      return null;
    }

    setLoading(true);
    setError(null);

    try {
      const program = getProgram(connection, {
        publicKey,
        signTransaction,
      });
      
      // Get the escrow PDA using payer's public key
      const [escrowPda] = await getEscrowPDA(
        manuscriptId,
        new PublicKey(payerPublicKey)
      );
      
      // Get the escrow token account PDA
      const escrowTokenAccountPda = await getEscrowTokenAccountPDA(escrowPda);
      
      // Get the publisher's USDC token account
      const publisherTokenAccount = await getAssociatedTokenAddress(
        USDC_MINT,
        publicKey
      );

      // Release the funds
      const tx = await program.methods
        .release()
        .accounts({
          escrow: escrowPda,
          escrowTokenAccount: escrowTokenAccountPda,
          recipientTokenAccount: publisherTokenAccount,
          publisher: publicKey,
          tokenProgram: TOKEN_PROGRAM_ID,
        })
        .rpc();

      return {
        transactionSignature: tx,
      };
    } catch (err: any) {
      setError(`Failed to release escrow funds: ${err.message}`);
      console.error("Error releasing escrow funds:", err);
      return null;
    } finally {
      setLoading(false);
    }
  };

  // Refund the author if manuscript is rejected
  const refundEscrow = async (manuscriptId: number, payerPublicKey: string) => {
    if (!publicKey || !signTransaction) {
      setError("Wallet not connected");
      return null;
    }

    setLoading(true);
    setError(null);

    try {
      const program = getProgram(connection, {
        publicKey,
        signTransaction,
      });
      
      // Get the escrow PDA using payer's public key
      const [escrowPda] = await getEscrowPDA(
        manuscriptId,
        new PublicKey(payerPublicKey)
      );
      
      // Get the escrow token account PDA
      const escrowTokenAccountPda = await getEscrowTokenAccountPDA(escrowPda);
      
      // Get the payer's USDC token account
      const payerTokenAccount = await getAssociatedTokenAddress(
        USDC_MINT,
        new PublicKey(payerPublicKey)
      );

      // Refund the escrow
      const tx = await program.methods
        .refund()
        .accounts({
          escrow: escrowPda,
          escrowTokenAccount: escrowTokenAccountPda,
          payerTokenAccount: payerTokenAccount,
          refundAuthority: publicKey,
          tokenProgram: TOKEN_PROGRAM_ID,
        })
        .rpc();

      return {
        transactionSignature: tx,
      };
    } catch (err: any) {
      setError(`Failed to refund escrow: ${err.message}`);
      console.error("Error refunding escrow:", err);
      return null;
    } finally {
      setLoading(false);
    }
  };

  return {
    initializeEscrow,
    fundEscrow,
    approveEscrow,
    releaseEscrow,
    refundEscrow,
    loading,
    error
  };
}
```

## Utility Functions

### IPFS Storage Utility
File: `client/src/utils/ipfsStorage.ts`

```typescript
import { Web3Storage } from 'web3storage';
import { create } from 'ipfs-http-client';
import { CID } from 'multiformats/cid';

// Load API key from environment
const WEB3_STORAGE_TOKEN = import.meta.env.VITE_WEB3_STORAGE_API_KEY;

// Initialize Web3Storage client
let web3StorageClient: Web3Storage | null = null;

function getWeb3StorageClient() {
  if (!web3StorageClient) {
    if (!WEB3_STORAGE_TOKEN) {
      throw new Error('Web3Storage API key is required');
    }
    web3StorageClient = new Web3Storage({ token: WEB3_STORAGE_TOKEN });
  }
  return web3StorageClient;
}

// Connect to IPFS via HTTP client (fallback option)
const ipfsClient = create({
  host: 'ipfs.infura.io',
  port: 5001,
  protocol: 'https'
});

/**
 * Store a file or blob on IPFS using Web3Storage
 * @param file The file or blob to store
 * @param onProgress Optional callback for upload progress
 * @returns CID of the stored content
 */
export async function storeFile(
  file: File | Blob,
  onProgress?: (progress: number) => void
): Promise<string> {
  try {
    // First try using Web3Storage
    const client = getWeb3StorageClient();
    const fileName = file instanceof File ? file.name : 'blob';
    const fileObj = file instanceof File ? file : new File([file], fileName);
    
    // Upload with progress tracking
    const cid = await client.put([fileObj], {
      onRootCidReady: (rootCid) => {
        console.log('Root CID:', rootCid);
        onProgress?.(10); // Initial progress
      },
      onStoredChunk: (chunkSize) => {
        console.log(`Stored chunk of size ${chunkSize}`);
        onProgress?.(50); // Mid progress
      }
    });
    
    onProgress?.(100); // Complete
    return cid;
  } catch (error) {
    console.error("Error storing with Web3Storage:", error);
    
    // Fallback to IPFS HTTP client
    try {
      const result = await ipfsClient.add(file);
      return result.path;
    } catch (ipfsError) {
      console.error("Error storing with IPFS HTTP client:", ipfsError);
      throw new Error("Failed to store file on IPFS");
    }
  }
}

/**
 * Store JSON metadata on IPFS
 * @param metadata The JSON metadata to store
 * @returns CID of the stored content
 */
export async function storeMetadata(metadata: any): Promise<string> {
  try {
    const metadataBlob = new Blob([JSON.stringify(metadata)], {
      type: 'application/json'
    });
    return await storeFile(metadataBlob);
  } catch (error) {
    console.error("Error storing metadata:", error);
    throw new Error("Failed to store metadata on IPFS");
  }
}

/**
 * Retrieves content from IPFS by CID
 * @param cid The content identifier
 * @returns The retrieved data as a Blob
 */
export async function retrieveFromIPFS(cid: string): Promise<Blob> {
  try {
    // First validate the CID
    try {
      CID.parse(cid);
    } catch (error) {
      throw new Error("Invalid CID format");
    }
    
    // Attempt to retrieve using Web3Storage
    const client = getWeb3StorageClient();
    const res = await client.get(cid);
    
    if (!res || !res.ok) {
      throw new Error("Failed to retrieve from Web3Storage");
    }
    
    const files = await res.files();
    if (!files || files.length === 0) {
      throw new Error("No files found in IPFS response");
    }
    
    return new Blob([await files[0].arrayBuffer()]);
  } catch (error) {
    console.error("Error retrieving from Web3Storage:", error);
    
    // Fallback to direct gateway fetch
    try {
      const response = await fetch(`https://ipfs.io/ipfs/${cid}`);
      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }
      return await response.blob();
    } catch (gatewayError) {
      console.error("Error retrieving from IPFS gateway:", gatewayError);
      throw new Error("Failed to retrieve content from IPFS");
    }
  }
}

/**
 * Formats an IPFS CID as a proper URI
 * @param cid The content identifier
 * @returns Formatted IPFS URI
 */
export function formatIPFSUri(cid: string): string {
  if (!cid) return '';
  
  // Check if already formatted
  if (cid.startsWith('ipfs://')) {
    return cid;
  }
  
  return `ipfs://${cid}`;
}

/**
 * Converts an IPFS URI to an HTTP gateway URL for browser display
 * @param ipfsUri The IPFS URI (ipfs://...)
 * @returns HTTP gateway URL
 */
export function ipfsUriToGatewayUrl(ipfsUri: string): string {
  if (!ipfsUri) return '';
  
  // Extract CID
  const cid = ipfsUri.replace('ipfs://', '');
  
  // Use multiple gateways for redundancy
  const gateways = [
    `https://ipfs.io/ipfs/${cid}`,
    `https://cloudflare-ipfs.com/ipfs/${cid}`,
    `https://gateway.pinata.cloud/ipfs/${cid}`,
    `https://dweb.link/ipfs/${cid}`,
  ];
  
  // Return primary gateway
  return gateways[0];
}
```

This compilation provides a comprehensive overview of the key components and architecture of the Fronsciers platform. The code demonstrates how the system leverages blockchain technology, particularly the Solana ecosystem, to create a decentralized academic publishing platform with features like DOCI generation, peer review, and badge issuance.