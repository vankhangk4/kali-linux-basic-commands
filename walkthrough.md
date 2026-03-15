# Báo Cáo Test Frontend - Academic Misconduct App

## Tổng quan
Đã thực hiện kiểm thử frontend toàn diện bao gồm: test trực tiếp trên trình duyệt, review code, và phân tích logic cho ứng dụng Academic Misconduct.

**Tài khoản test:** `staff@gmail.com` / `123456` (Admin/Staff), `student@gmail.com` / `123456` (Student)

---

## ✅ Các trang đã test thành công

### Auth Pages
| Trang | UI | Responsive | Flow | Kết quả |
|-------|-----|-----------|------|---------|
| Login | ✅ Sạch, chuyên nghiệp | ✅ Mobile 375px OK | ✅ Validation, redirect theo role | **PASS** |
| Forgot Password | ✅ Form rõ ràng | ✅ | ✅ Gửi email reset | **PASS** |
| Change Password | ✅ | ✅ | ✅ Validate mật khẩu mới | **PASS** |

### Staff Pages
| Trang | UI | Flow | Kết quả |
|-------|-----|------|---------|
| Dashboard | ✅ Banner, courses, assignments | ✅ Navigation hoạt động | **PASS** |
| Courses | ✅ Danh sách khóa học | ✅ Click vào course | **PASS** |
| Statistics | ✅ Bảng thống kê, pie chart, bar chart | ✅ Filter, sort | **PASS** |
| Profile Dropdown | ✅ Menu hiển thị đúng | ✅ | **PASS** |

### Student Pages
| Trang | UI | Flow | Kết quả |
|-------|-----|------|---------|
| Dashboard | ✅ Courses, upcoming, stats chart | ✅ Navigation | **PASS** |
| History | ✅ Action required, submission table | ✅ Search, filter, sort | **PASS** |

````carousel
![Login Desktop](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/login_page_desktop_1773556452996.png)
<!-- slide -->
![Staff Dashboard](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/staff_dashboard_1773556510676.png)
<!-- slide -->
![Student Dashboard](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/student_dashboard_1773556612586.png)
<!-- slide -->
![Statistics](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/staff_statistics_page_1773556557313.png)
````

---

## 🐛 Các Bug Đã Phát Hiện

### Bug 1: Lỗi Build nghiêm trọng — [DataService.ts](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/services/DataService.ts) (ĐÃ SỬA ✅)
- **Mức độ:** 🔴 Critical
- **File:** [DataService.ts](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/services/DataService.ts#L528-L540)
- **Mô tả:** Hàm [updateSubmission](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/services/DataService.ts#528-593) có tham số `fileUrl` bị khai báo **trùng lặp** 2 lần, gây lỗi TypeScript build và app không thể chạy.
- **Trạng thái:** ✅ Đã sửa

```diff:DataService.ts
import {addDoc,deleteDoc, setDoc, doc, getDoc, updateDoc, collection, getDocs, query, where, arrayRemove } from "firebase/firestore";
import { initializeApp } from "firebase/app";
import { getFirestore } from "firebase/firestore";
import {getStorage, ref, listAll, deleteObject} from "firebase/storage";
import { evaluateEnvironmentForUser } from "./loginEnvironmentService";

// Firebase config
const firebaseConfig = {
  apiKey: "AIzaSyALK_A608G1BLvkPM_F2TiYatrhLi1_5WE",
  authDomain: "academic-misconduct-database.firebaseapp.com",
  projectId: "academic-misconduct-database",
  storageBucket: "academic-misconduct-database.firebasestorage.app",
  messagingSenderId: "1024421990474",
  appId: "1:1024421990474:web:551f543208be4f343256b3"
};

export const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);

// ---------------- TYPES ---------------- //
export interface User {
  uid: string;
  userId: string;
  name: string;
  role: "student" | "staff" | "admin";
  email: string;
  courses: string[];
}

export interface Assignment {
  id: string;
  title: string;
  openedTime: string;
  dueTime: string;
}

export interface Exercise {
  id: string;
  title: string;
  openedTime: string;
  dueTime: string;
}

export interface LearningExpectation {
  id: string;
  title: string;
  openedTime: string;
  dueTime: string;
  questionDescription: string;
}

export interface FileMetadata {
  fileName: string;
  fileSize: number;
  fileType: string;
  dateCreated: string; // ISO string
  dateModified: string; // ISO string
  uploadedAt: string; // ISO string - when uploaded to system
}

export interface SubmissionStatus {
  nameOfStudent?:  string,
  nameOfSubmission?: string,
  submittedAt?: string;
  fileUrl?: string;
  cheatingStatus: boolean[];
  fileMetadata?: FileMetadata;
  status: "Submitted" | "Late" | "Missing" | "Cheated" | "Regraded";
  score?: number;
  evaluation?: string;
  miniQuizStatus: boolean;
  miniQuizQuestion: string | null;
  miniQuizAnswer: string | null;
}

export interface SubmissionWithUser {
  uid: string;
  submission: SubmissionStatus;
}

export interface Course {
  courseId: string;
  code: string;
  name: string;
  staffId: string;
  studentUIDs: string[]; 
  description: string;
  startDate: string;
  endDate: string;
  assignments: Record<string, Assignment>;
  exercises: Record<string, Exercise>;
  learningExpectations: Record<string, LearningExpectation>;
  submissions?: Record<string, Record<string, SubmissionStatus>>;
}

// ===== DELETE FOLDER AND CONTENTS (HELPER - OUTSIDE CLASS) =====
const storage = getStorage();

async function deleteFolder(folderPath: string): Promise<void> {
  const folderRef = ref(storage, folderPath);

  try {
    const res = await listAll(folderRef);

    // delete files
    await Promise.all(
      res.items.map(item => deleteObject(item))
    );

    // delete subfolders (recursive)
    await Promise.all(
      res.prefixes.map(sub =>
        deleteFolder(sub.fullPath)
      )
    );
  } catch (err) {
    console.warn("Delete folder failed:", folderPath, err);
  }
}

// ---------------- SERVICE ---------------- //
class DataService {
  // -------- GET ALL COURSES --------
  async getAllCourses(): Promise<Course[]> {
    const snap = await getDocs(collection(db, "courses"));
    const courses: Course[] = [];
    snap.forEach((docSnap) => {
      courses.push({ courseId: docSnap.id, ...(docSnap.data() as Omit<Course, "courseId">) });
    });
    return courses;
  }

  // -------- GET ALL USERS --------
  async getAllUsers(): Promise<User[]> {
    const snap = await getDocs(collection(db, "users"));
    const users: User[] = [];
    snap.forEach((docSnap) => {
      const data = docSnap.data();
      users.push({
        uid: docSnap.id,
        userId: data.userId || "",
        name: data.name || "",
        role: (data.role as "student" | "staff" | "admin") || "student",
        email: data.email || "",
        courses: data.courses || [],
      });
    });
    return users;
  }

  // -------- UPDATE USER --------
  async updateUser(uid: string, data: Partial<User>) {
    try {
      const userRef = doc(db, "users", uid);
      // Check if document exists first
      const userSnap = await getDoc(userRef);
      if (!userSnap.exists()) {
        throw new Error(`User with UID ${uid} does not exist`);
      }
      // Use setDoc with merge instead of updateDoc to work better with security rules
      await setDoc(userRef, data, { merge: true });
      return true;
    } catch (error) {
      console.error("Error updating user:", error);
      return false;
    }
  }

  // -------- DELETE USER --------
  async deleteUser(uid: string) {
    try {
      await deleteDoc(doc(db, "users", uid));
      return true;
    } catch (error) {
      console.error("Error deleting user:", error);
      return false;
    }
  }

  // -------- GET SINGLE COURSE --------
  async getCourse(courseId: string): Promise<Course | null> {
    const snap = await getDoc(doc(db, "courses", courseId));

    return snap.exists() ? ({ courseId: snap.id, ...(snap.data() as Omit<Course, "courseId">) }) : null;
  }

  // -------- UPDATE USER COURSES --------
  async updateUserCourses(userId: string, courseId: string) {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const userRef = doc(db, "users", uid);
    const snap = await getDoc(userRef);

    if (!snap.exists()) {
      throw new Error(`User ${uid} does not exist`);
    }

    const data = snap.data() as User;
    const courses = data.courses || [];

    if (courses.includes(courseId)) return; 

    await updateDoc(userRef, {
      courses: [...courses, courseId],
    });
  }

    // -------- GET SINGLE SUBMISSION --------
async getSubmission(
  courseId: string,
  submissionId: string,
  userId: string
): Promise<SubmissionStatus | null> {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );

    const snap = await getDoc(ref);

    if (!snap.exists()) return null;

    const data = snap.data() as Partial<SubmissionStatus>;

    const submission: SubmissionStatus = {
      nameOfStudent: data.nameOfStudent,
      nameOfSubmission: data.nameOfSubmission,
      submittedAt: data.submittedAt,
      fileUrl: data.fileUrl,

      cheatingStatus: data.cheatingStatus ?? [],

      status: data.status ?? "Missing",

      score: data.score ?? 0,
      evaluation: data.evaluation ?? "",

      miniQuizStatus: data.miniQuizStatus ?? false,
      miniQuizQuestion: data.miniQuizQuestion ?? null,
      miniQuizAnswer: data.miniQuizAnswer ?? null,
    };

    return submission;
  } catch (error) {
    console.error("Error fetching submission:", error);
    return null;
  }
}

  // ===== ADD ASSIGNMENT =====
  async addAssignment(courseId: string, data: Omit<Assignment, "id">) {
    const ref = await addDoc(
      collection(db, "courses", courseId, "assignments"),
      data
    );
    return { id: ref.id, ...data };
  }

  // ===== ADD EXERCISE =====
  async addExercise(courseId: string, data: Omit<Exercise, "id">) {
    const ref = await addDoc(
      collection(db, "courses", courseId, "exercises"),
      data
    );
    return { id: ref.id, ...data };
  }

  // ===== ADD LEARNING EXPECTATION =====
  async addLearningExpectation(
    courseId: string,
    data: Omit<LearningExpectation, "id">
  ) {
    const ref = await addDoc(
      collection(db, "courses", courseId, "learningExpectations"),
      data
    );
    return { id: ref.id, ...data };
  }

  async deleteAssignment(courseId: string, id: string) {
    await deleteDoc(doc(db, "courses", courseId, "assignments", id));
  }

  async deleteExercise(courseId: string, id: string) {
    await deleteDoc(doc(db, "courses", courseId, "exercises", id));
  }

  async deleteLearningExpectation(courseId: string, id: string) {
    await deleteDoc(
      doc(db, "courses", courseId, "learningExpectations", id)
    );
  }



  // -------- LIST ALL SUBMISSIONS FOR A SUBMISSION ITEM --------
  // Reads: courses/{courseId}/submissions/{submissionId}/users/*
  async listSubmissionsForItem(
    courseId: string,
    submissionId: string
  ): Promise<SubmissionWithUser[]> {
    try {
      const usersRef = collection(
        db,
        "courses",
        courseId,
        "submissions",
        submissionId,
        "users"
      );

      const snap = await getDocs(usersRef);

      const results: SubmissionWithUser[] = snap.docs.map((d) => {
        const data = d.data() as Partial<SubmissionStatus>;
        const submission: SubmissionStatus = {
          nameOfStudent: data.nameOfStudent,
          nameOfSubmission: data.nameOfSubmission,
          submittedAt: data.submittedAt,
          fileUrl: data.fileUrl,
          cheatingStatus: data.cheatingStatus ?? [],
          status: data.status ?? "Missing",
          score: data.score ?? 0,
          evaluation: data.evaluation ?? "",
          miniQuizStatus: data.miniQuizStatus ?? false,
          miniQuizQuestion: data.miniQuizQuestion ?? null,
          miniQuizAnswer: data.miniQuizAnswer ?? null,
        };
        return { uid: d.id, submission };
      });

      return results;
    } catch (error) {
      console.error("Error listing submissions:", error);
      return [];
    }
  }
  // -------- COUNT STUDENTS IN A COURSE --------
  // Uses users collection: query by courses array-contains courseId, then filters role === "student"
  async getStudentCountForCourse(courseId: string): Promise<number | null> {
    try {
      const usersRef = collection(db, "users");
      const q = query(usersRef, where("courses", "array-contains", courseId));
      const snap = await getDocs(q);
      const users = snap.docs.map((d) => d.data() as Partial<User>);
      return users.filter((u) => u.role === "student").length;
    } catch (error) {
      console.error("Error counting students for course:", error);
      return null;
    }
  }

  // -------- LIST STUDENTS IN A COURSE --------
  // Uses users collection: query by courses array-contains courseId, then filters role === "student"
 async listStudentsForCourse(courseId: string): Promise<User[]> {
  try {
    const usersRef = collection(db, "users");
    const q = query(usersRef, where("courses", "array-contains", courseId));
    const snap = await getDocs(q);

    const users: User[] = snap.docs
      .map((d) => {
        const data = d.data() as User;

        return {
          uid: d.id,                
          userId: data.userId,      
          name: data.name,
          role: data.role,
          email: data.email,
          courses: data.courses || [],
        };
      })
      .filter((u) => u.role === "student");

    return users;
  } catch (error) {
    console.error("Error listing students for course:", error);
    return [];
  }
}

  // -------- GET USER --------
  async getUser(uid: string): Promise<User | null> {
    // 1. Try by document ID (uid)
    const snap = await getDoc(doc(db, "users", uid));
    if (snap.exists()) {
      return snap.data() as User;
    }

    // 2. Try by 'uid' field
    const qUid = query(collection(db, "users"), where("uid", "==", uid));
    const snapUid = await getDocs(qUid);
    if (!snapUid.empty) {
      return { uid: snapUid.docs[0].id, ...snapUid.docs[0].data() } as User;
    }

    // 3. Try by 'userId' field (legacy/alternative ID)
    const qUserId = query(collection(db, "users"), where("userId", "==", uid));
    const snapUserId = await getDocs(qUserId);
    if (!snapUserId.empty) {
       return { uid: snapUserId.docs[0].id, ...snapUserId.docs[0].data() } as User;
    }

    return null;
  }

  // -------- GET USER BY EMAIL --------
  async getUserByEmail(email: string): Promise<User | null> {
    try {
      const q = query(
        collection(db, "users"),
        where("email", "==", email.toLowerCase().trim())
      );
      const snap = await getDocs(q);
      
      if (!snap.empty) {
        return { uid: snap.docs[0].id, ...snap.docs[0].data() } as User;
      }
      return null;
    } catch (error) {
      console.error("Error getting user by email:", error);
      return null;
    }
  }
  
  async getUidByUserId(userId: string): Promise<string | null> {
    const q = query(
      collection(db, "users"),
      where("userId", "==", userId)
    );

    const snap = await getDocs(q);

    if (snap.empty) return null;

    return snap.docs[0].id; 
  }
  // -------- GET ASSIGNMENTS --------
async getAssignments(courseId: string): Promise<Assignment[]> {
  const assignmentsRef = collection(db, "courses", courseId, "assignments");
  const snapshot = await getDocs(assignmentsRef);
  const assignments = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })) as Assignment[];
  return assignments;
}

// -------- GET EXERCISES --------
async getExercises(courseId: string): Promise<Exercise[]> {
  const exercisesRef = collection(db, "courses", courseId, "exercises");
  const snapshot = await getDocs(exercisesRef);
  const exercises = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })) as Exercise[];
  return exercises;
}

// -------- GET LEARNING EXPECTATIONS --------
async getLearningExpectations(courseId: string): Promise<LearningExpectation[]> {
  const expectationsRef = collection(db, "courses", courseId, "learningExpectations");
  const snapshot = await getDocs(expectationsRef);
  const expectations = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })) as LearningExpectation[];
  return expectations;
}


  // -------- UPDATE LEARNING EXPECTATION SUBMISSION --------
  async updateLearningExpectation(
  courseId: string,
  submissionId: string,
  userId: string,
  answer: string
) {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );

    const envCheck = evaluateEnvironmentForUser(userId);
    const userData = await dataService.getUser(uid);
    const courseData = await dataService.getCourse(courseId);

    const submissionObj = {
      nameOfStudent:  userData?.name || "Unknown Student",
      nameOfSubmission: courseData?.name || "Unknown Submission",
      status: "Submitted",
      submittedAt: new Date().toISOString(),
      fileUrl: answer, 
      // Index 0 now represents abnormal login environment (location/device/time)
      cheatingStatus: [envCheck.isAbnormal, false, false],
      score: null,
      evaluation: null,
      miniQuizStatus: false,
      miniQuizQuestion: null,
      miniQuizAnswer: null,
    };

    await setDoc(ref, submissionObj, { merge: true });

    console.log("Updated successfully");
    return true;

  } catch (error) {
    console.error("Error update learning expectation:", error);
    return null;
  }
}

// -------- UPDATE SUBMISSION --------
//to do: update cheating status
async updateSubmission(
  courseId: string,
  submissionId: string,
  userId: string,
  fileUrl: string,
  plagiarismLabel: 0 | 1 = 0
  fileUrl?: string,
  score?: number,
  evaluation?: string
) {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${userId}`);
    }

    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );
    
if (typeof fileUrl === "string" && fileUrl.length > 0) {
  const userData = await dataService.getUser(uid);
  const courseData = await dataService.getCourse(courseId);

  const hasPlagiarismSignal = plagiarismLabel === 1;
  const cheatingStatus: boolean[] = [false, hasPlagiarismSignal, false];
  const status = hasPlagiarismSignal ? "Cheated" : "Submitted";

  const submissionObj = {
    nameOfStudent: userData?.name || "Unknown Student",
    nameOfSubmission: courseData?.name || "Unknown Submission",
    status,
    submittedAt: new Date().toISOString(),
    fileUrl: fileUrl,
    cheatingStatus,
    score: null,
    evaluation: null,
    miniQuizStatus: false,
    miniQuizQuestion: null,
    miniQuizAnswer: null,
  };

  await setDoc(ref, submissionObj, { merge: true });

} else {
  await updateDoc(ref, {
    score: score ?? null,
    evaluation: evaluation ?? null,
  });
}
    console.log("Updated successfully");
    return true;

  } catch (error) {
    console.error("Error updating submission:", error);
    return null;
  }
}
// -------- GET USERS BY COURSE --------
async getUsersByCourse(courseId: string): Promise<User[]> {
  try {
    const usersRef = collection(db, "users");
    const q = query(usersRef, where("courses", "array-contains", courseId));
    const snap = await getDocs(q);
    
    const users: User[] = snap.docs.map(docSnap => {
      const userData = docSnap.data() as Partial<User>;
      return {
        uid: docSnap.id,               
        userId: userData.userId || "", 
        name: userData.name || "",
        role: userData.role as "student" | "staff" | "admin",
        email: userData.email || "",
        courses: userData.courses || [],
      };
    }).filter(u => u.role === "student"); 
    
    console.log(`Found ${users.length} users in course ${courseId}:`, users);
    return users;
    
  } catch (error) {
    console.error("Error getting users by course:", error);
    return [];
  }
}


// -------- GET ALL SUBMISSIONS FOR AN ITEM --------
async getSubmissionsForLearningExpectation(
  courseId: string,
  submissionId: string
): Promise<Record<string, SubmissionStatus>> {
  try {
    const submissionsRef = collection(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users"
    );
    
    const snap = await getDocs(submissionsRef);
    const submissions: Record<string, SubmissionStatus> = {};
    
    snap.forEach((docSnap) => {
      submissions[docSnap.id] = docSnap.data() as SubmissionStatus;
    });
    
    return submissions;
  } catch (error) {
    console.error("Error fetching submissions:", error);
    return {};
  }
}
  // -------- UPDATE MINI QUIZ SUBMISSION --------
  async updateSubmissionMiniQuiz(
  courseId: string,
  submissionId: string,
  userId: string,
  question: string,
  answer: string
) {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${userId}`);
    }
    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );

    const snap = await getDoc(ref);

    if (!snap.exists()) return null;

    await updateDoc(ref, {
      miniQuizQuestion: question,
      miniQuizAnswer: answer,
      miniQuizStatus: true,
    });

    return true;
  } catch (error) {
    console.error("Error update mini quiz:", error);
    return null;
  }
}


  // -------- TEACHER REGRADES MINI QUIZ --------
  async updateSubmissionMiniQuizRegrade(courseId: string, submissionId: string, userId: string) {
    try {
         const uid = await this.getUidByUserId(userId);
        if (!uid) {
          throw new Error(`No UID found for userId: ${userId}`);
        }
        const ref = doc(
          db,
          "courses",
          courseId,
          "submissions",
          submissionId,
          "users",
          uid
        );

        const snap = await getDoc(ref);

        if (!snap.exists()) return null;

        await updateDoc(ref, {
          status :"Regraded",
          miniQuizStatus: false,
          miniQuizQuestion:null,
          miniQuizAnswer :null
        });

        return true;
      } catch (error) {
        console.error("Error regard mini quiz:", error);
        return null;
      }
  }


   async addCourse(course: {
    code: string;
    name: string;
    staffId: string;
    studentUIDs: string[];
    description: string;
    startDate: string;
    endDate: string;
  }) {
    // ===== CHECK DATE =====
    const start = new Date(course.startDate).getTime();
    const end = new Date(course.endDate).getTime();
    const now = Date.now();

    if (isNaN(start) || isNaN(end)) {
      throw new Error("Invalid date format");
    }

    if (start >= end) {
      throw new Error("Start date must be before end date");
    }

    if (end <= now) {
      throw new Error("End date must be in the future");
    }

    // ===== CHECK COURSE CODE EXISTS =====
    const courseQuery = query(
      collection(db, "courses"),
      where("code", "==", course.code)
    );

    const courseSnap = await getDocs(courseQuery);

    if (!courseSnap.empty) {
      throw new Error("Course code already exists");
    }

    // ===== CHECK STAFF EXISTS & ROLE =====
    const uid = await this.getUidByUserId(course.staffId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const staffRef = doc(db, "users", uid);
    const staffSnap = await getDoc(staffRef);

    if (!staffSnap.exists()) {
      throw new Error("Staff ID does not exist");
    }

    const staffData = staffSnap.data();

    if (staffData.role !== "staff") {
      throw new Error("User is not a staff");
    }

    // ===== CHECK STUDENT IDS =====
    if (!course.studentUIDs || course.studentUIDs.length === 0) {
      throw new Error("Student IDs are required");
    }

    const studentUIDs: string[] = [];

    for (const studentId of course.studentUIDs) {
      const uid = await this.getUidByUserId(studentId);
      if (!uid) {
        throw new Error(`No UID found for userId: ${studentId}`);
      }

      const studentRef = doc(db, "users", uid);
      const studentSnap = await getDoc(studentRef);

      if (!studentSnap.exists()) {
        throw new Error(`Student with UID ${uid} does not exist`);
      }

      const studentData = studentSnap.data() as User;

      if (studentData.role !== "student") {
        throw new Error(`User ${studentId} (UID: ${uid}) is not a student`);
      }

      studentUIDs.push(uid);
    }

    // ===== CREATE COURSE =====
    const docRef = await addDoc(collection(db, "courses"), {
      code: course.code,
      name: course.name,
      staffId: course.staffId,
      studentUIDs: studentUIDs, 
      description: course.description,
      startDate: course.startDate,
      endDate: course.endDate,
      assignments: {},
      exercises: {},
      learningExpectations: {},
      createdAt: new Date().toISOString(),
    });

    return docRef.id;
  }

    // -------- REMOVE COURSE FROM USER --------
  async removeCourseFromUser(userId: string, courseId: string) {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${userId}`);
    }
    const userRef = doc(db, "users", uid);
    const snap = await getDoc(userRef);

    if (!snap.exists()) return;

    const data = snap.data() as User;

    const updatedCourses = (data.courses || []).filter(
      (id) => id !== courseId
    );

    await updateDoc(userRef, {
      courses: updatedCourses,
    });
  }

  // ===== DELETE COURSE (FULL CLEAN) =====
  async deleteCourseAndCleanup(courseId: string) {
    const courseRef = doc(db, "courses", courseId);
    const courseSnap = await getDoc(courseRef);

    if (!courseSnap.exists()) {
      throw new Error("Course not found");
    }

    const course = courseSnap.data() as Course;

    const staffUid = await dataService.getUidByUserId(course.staffId);

    if (!staffUid) {
      throw new Error("Staff UID not found");
    }

    const uids = [
      staffUid,
      ...(course.studentUIDs || []),
    ];

    await Promise.all(
    uids.map((uid) =>
      updateDoc(doc(db, "users", uid), {
        courses: arrayRemove(courseId),
      })
      )
    );

    await deleteFolder(courseId);
    await deleteDoc(courseRef);
  }

  // -------- UPDATE COURSE --------
  async updateCourse(courseId: string, data: Partial<Course>) {
    try {
      const courseRef = doc(db, "courses", courseId);
      // Check if document exists first
      const courseSnap = await getDoc(courseRef);
      if (!courseSnap.exists()) {
        throw new Error(`Course with ID ${courseId} does not exist`);
      }
      // Use updateDoc to work better with security rules
      await setDoc(courseRef, data, { merge: true });
      return true;
    } catch (error) {
      console.error("Error updating course:", error);
      return false;
    }
  }
}

export const dataService = new DataService();
===
import {addDoc,deleteDoc, setDoc, doc, getDoc, updateDoc, collection, getDocs, query, where, arrayRemove } from "firebase/firestore";
import { initializeApp } from "firebase/app";
import { getFirestore } from "firebase/firestore";
import {getStorage, ref, listAll, deleteObject} from "firebase/storage";
import { evaluateEnvironmentForUser } from "./loginEnvironmentService";

// Firebase config
const firebaseConfig = {
  apiKey: "AIzaSyALK_A608G1BLvkPM_F2TiYatrhLi1_5WE",
  authDomain: "academic-misconduct-database.firebaseapp.com",
  projectId: "academic-misconduct-database",
  storageBucket: "academic-misconduct-database.firebasestorage.app",
  messagingSenderId: "1024421990474",
  appId: "1:1024421990474:web:551f543208be4f343256b3"
};

export const app = initializeApp(firebaseConfig);
export const db = getFirestore(app);

// ---------------- TYPES ---------------- //
export interface User {
  uid: string;
  userId: string;
  name: string;
  role: "student" | "staff" | "admin";
  email: string;
  courses: string[];
}

export interface Assignment {
  id: string;
  title: string;
  openedTime: string;
  dueTime: string;
}

export interface Exercise {
  id: string;
  title: string;
  openedTime: string;
  dueTime: string;
}

export interface LearningExpectation {
  id: string;
  title: string;
  openedTime: string;
  dueTime: string;
  questionDescription: string;
}

export interface FileMetadata {
  fileName: string;
  fileSize: number;
  fileType: string;
  dateCreated: string; // ISO string
  dateModified: string; // ISO string
  uploadedAt: string; // ISO string - when uploaded to system
}

export interface SubmissionStatus {
  nameOfStudent?:  string,
  nameOfSubmission?: string,
  submittedAt?: string;
  fileUrl?: string;
  cheatingStatus: boolean[];
  fileMetadata?: FileMetadata;
  status: "Submitted" | "Late" | "Missing" | "Cheated" | "Regraded";
  score?: number;
  evaluation?: string;
  miniQuizStatus: boolean;
  miniQuizQuestion: string | null;
  miniQuizAnswer: string | null;
}

export interface SubmissionWithUser {
  uid: string;
  submission: SubmissionStatus;
}

export interface Course {
  courseId: string;
  code: string;
  name: string;
  staffId: string;
  studentUIDs: string[]; 
  description: string;
  startDate: string;
  endDate: string;
  assignments: Record<string, Assignment>;
  exercises: Record<string, Exercise>;
  learningExpectations: Record<string, LearningExpectation>;
  submissions?: Record<string, Record<string, SubmissionStatus>>;
}

// ===== DELETE FOLDER AND CONTENTS (HELPER - OUTSIDE CLASS) =====
const storage = getStorage();

async function deleteFolder(folderPath: string): Promise<void> {
  const folderRef = ref(storage, folderPath);

  try {
    const res = await listAll(folderRef);

    // delete files
    await Promise.all(
      res.items.map(item => deleteObject(item))
    );

    // delete subfolders (recursive)
    await Promise.all(
      res.prefixes.map(sub =>
        deleteFolder(sub.fullPath)
      )
    );
  } catch (err) {
    console.warn("Delete folder failed:", folderPath, err);
  }
}

// ---------------- SERVICE ---------------- //
class DataService {
  // -------- GET ALL COURSES --------
  async getAllCourses(): Promise<Course[]> {
    const snap = await getDocs(collection(db, "courses"));
    const courses: Course[] = [];
    snap.forEach((docSnap) => {
      courses.push({ courseId: docSnap.id, ...(docSnap.data() as Omit<Course, "courseId">) });
    });
    return courses;
  }

  // -------- GET ALL USERS --------
  async getAllUsers(): Promise<User[]> {
    const snap = await getDocs(collection(db, "users"));
    const users: User[] = [];
    snap.forEach((docSnap) => {
      const data = docSnap.data();
      users.push({
        uid: docSnap.id,
        userId: data.userId || "",
        name: data.name || "",
        role: (data.role as "student" | "staff" | "admin") || "student",
        email: data.email || "",
        courses: data.courses || [],
      });
    });
    return users;
  }

  // -------- UPDATE USER --------
  async updateUser(uid: string, data: Partial<User>) {
    try {
      const userRef = doc(db, "users", uid);
      // Check if document exists first
      const userSnap = await getDoc(userRef);
      if (!userSnap.exists()) {
        throw new Error(`User with UID ${uid} does not exist`);
      }
      // Use setDoc with merge instead of updateDoc to work better with security rules
      await setDoc(userRef, data, { merge: true });
      return true;
    } catch (error) {
      console.error("Error updating user:", error);
      return false;
    }
  }

  // -------- DELETE USER --------
  async deleteUser(uid: string) {
    try {
      await deleteDoc(doc(db, "users", uid));
      return true;
    } catch (error) {
      console.error("Error deleting user:", error);
      return false;
    }
  }

  // -------- GET SINGLE COURSE --------
  async getCourse(courseId: string): Promise<Course | null> {
    const snap = await getDoc(doc(db, "courses", courseId));

    return snap.exists() ? ({ courseId: snap.id, ...(snap.data() as Omit<Course, "courseId">) }) : null;
  }

  // -------- UPDATE USER COURSES --------
  async updateUserCourses(userId: string, courseId: string) {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const userRef = doc(db, "users", uid);
    const snap = await getDoc(userRef);

    if (!snap.exists()) {
      throw new Error(`User ${uid} does not exist`);
    }

    const data = snap.data() as User;
    const courses = data.courses || [];

    if (courses.includes(courseId)) return; 

    await updateDoc(userRef, {
      courses: [...courses, courseId],
    });
  }

    // -------- GET SINGLE SUBMISSION --------
async getSubmission(
  courseId: string,
  submissionId: string,
  userId: string
): Promise<SubmissionStatus | null> {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );

    const snap = await getDoc(ref);

    if (!snap.exists()) return null;

    const data = snap.data() as Partial<SubmissionStatus>;

    const submission: SubmissionStatus = {
      nameOfStudent: data.nameOfStudent,
      nameOfSubmission: data.nameOfSubmission,
      submittedAt: data.submittedAt,
      fileUrl: data.fileUrl,

      cheatingStatus: data.cheatingStatus ?? [],

      status: data.status ?? "Missing",

      score: data.score ?? 0,
      evaluation: data.evaluation ?? "",

      miniQuizStatus: data.miniQuizStatus ?? false,
      miniQuizQuestion: data.miniQuizQuestion ?? null,
      miniQuizAnswer: data.miniQuizAnswer ?? null,
    };

    return submission;
  } catch (error) {
    console.error("Error fetching submission:", error);
    return null;
  }
}

  // ===== ADD ASSIGNMENT =====
  async addAssignment(courseId: string, data: Omit<Assignment, "id">) {
    const ref = await addDoc(
      collection(db, "courses", courseId, "assignments"),
      data
    );
    return { id: ref.id, ...data };
  }

  // ===== ADD EXERCISE =====
  async addExercise(courseId: string, data: Omit<Exercise, "id">) {
    const ref = await addDoc(
      collection(db, "courses", courseId, "exercises"),
      data
    );
    return { id: ref.id, ...data };
  }

  // ===== ADD LEARNING EXPECTATION =====
  async addLearningExpectation(
    courseId: string,
    data: Omit<LearningExpectation, "id">
  ) {
    const ref = await addDoc(
      collection(db, "courses", courseId, "learningExpectations"),
      data
    );
    return { id: ref.id, ...data };
  }

  async deleteAssignment(courseId: string, id: string) {
    await deleteDoc(doc(db, "courses", courseId, "assignments", id));
  }

  async deleteExercise(courseId: string, id: string) {
    await deleteDoc(doc(db, "courses", courseId, "exercises", id));
  }

  async deleteLearningExpectation(courseId: string, id: string) {
    await deleteDoc(
      doc(db, "courses", courseId, "learningExpectations", id)
    );
  }



  // -------- LIST ALL SUBMISSIONS FOR A SUBMISSION ITEM --------
  // Reads: courses/{courseId}/submissions/{submissionId}/users/*
  async listSubmissionsForItem(
    courseId: string,
    submissionId: string
  ): Promise<SubmissionWithUser[]> {
    try {
      const usersRef = collection(
        db,
        "courses",
        courseId,
        "submissions",
        submissionId,
        "users"
      );

      const snap = await getDocs(usersRef);

      const results: SubmissionWithUser[] = snap.docs.map((d) => {
        const data = d.data() as Partial<SubmissionStatus>;
        const submission: SubmissionStatus = {
          nameOfStudent: data.nameOfStudent,
          nameOfSubmission: data.nameOfSubmission,
          submittedAt: data.submittedAt,
          fileUrl: data.fileUrl,
          cheatingStatus: data.cheatingStatus ?? [],
          status: data.status ?? "Missing",
          score: data.score ?? 0,
          evaluation: data.evaluation ?? "",
          miniQuizStatus: data.miniQuizStatus ?? false,
          miniQuizQuestion: data.miniQuizQuestion ?? null,
          miniQuizAnswer: data.miniQuizAnswer ?? null,
        };
        return { uid: d.id, submission };
      });

      return results;
    } catch (error) {
      console.error("Error listing submissions:", error);
      return [];
    }
  }
  // -------- COUNT STUDENTS IN A COURSE --------
  // Uses users collection: query by courses array-contains courseId, then filters role === "student"
  async getStudentCountForCourse(courseId: string): Promise<number | null> {
    try {
      const usersRef = collection(db, "users");
      const q = query(usersRef, where("courses", "array-contains", courseId));
      const snap = await getDocs(q);
      const users = snap.docs.map((d) => d.data() as Partial<User>);
      return users.filter((u) => u.role === "student").length;
    } catch (error) {
      console.error("Error counting students for course:", error);
      return null;
    }
  }

  // -------- LIST STUDENTS IN A COURSE --------
  // Uses users collection: query by courses array-contains courseId, then filters role === "student"
 async listStudentsForCourse(courseId: string): Promise<User[]> {
  try {
    const usersRef = collection(db, "users");
    const q = query(usersRef, where("courses", "array-contains", courseId));
    const snap = await getDocs(q);

    const users: User[] = snap.docs
      .map((d) => {
        const data = d.data() as User;

        return {
          uid: d.id,                
          userId: data.userId,      
          name: data.name,
          role: data.role,
          email: data.email,
          courses: data.courses || [],
        };
      })
      .filter((u) => u.role === "student");

    return users;
  } catch (error) {
    console.error("Error listing students for course:", error);
    return [];
  }
}

  // -------- GET USER --------
  async getUser(uid: string): Promise<User | null> {
    // 1. Try by document ID (uid)
    const snap = await getDoc(doc(db, "users", uid));
    if (snap.exists()) {
      return snap.data() as User;
    }

    // 2. Try by 'uid' field
    const qUid = query(collection(db, "users"), where("uid", "==", uid));
    const snapUid = await getDocs(qUid);
    if (!snapUid.empty) {
      return { uid: snapUid.docs[0].id, ...snapUid.docs[0].data() } as User;
    }

    // 3. Try by 'userId' field (legacy/alternative ID)
    const qUserId = query(collection(db, "users"), where("userId", "==", uid));
    const snapUserId = await getDocs(qUserId);
    if (!snapUserId.empty) {
       return { uid: snapUserId.docs[0].id, ...snapUserId.docs[0].data() } as User;
    }

    return null;
  }

  // -------- GET USER BY EMAIL --------
  async getUserByEmail(email: string): Promise<User | null> {
    try {
      const q = query(
        collection(db, "users"),
        where("email", "==", email.toLowerCase().trim())
      );
      const snap = await getDocs(q);
      
      if (!snap.empty) {
        return { uid: snap.docs[0].id, ...snap.docs[0].data() } as User;
      }
      return null;
    } catch (error) {
      console.error("Error getting user by email:", error);
      return null;
    }
  }
  
  async getUidByUserId(userId: string): Promise<string | null> {
    const q = query(
      collection(db, "users"),
      where("userId", "==", userId)
    );

    const snap = await getDocs(q);

    if (snap.empty) return null;

    return snap.docs[0].id; 
  }
  // -------- GET ASSIGNMENTS --------
async getAssignments(courseId: string): Promise<Assignment[]> {
  const assignmentsRef = collection(db, "courses", courseId, "assignments");
  const snapshot = await getDocs(assignmentsRef);
  const assignments = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })) as Assignment[];
  return assignments;
}

// -------- GET EXERCISES --------
async getExercises(courseId: string): Promise<Exercise[]> {
  const exercisesRef = collection(db, "courses", courseId, "exercises");
  const snapshot = await getDocs(exercisesRef);
  const exercises = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })) as Exercise[];
  return exercises;
}

// -------- GET LEARNING EXPECTATIONS --------
async getLearningExpectations(courseId: string): Promise<LearningExpectation[]> {
  const expectationsRef = collection(db, "courses", courseId, "learningExpectations");
  const snapshot = await getDocs(expectationsRef);
  const expectations = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })) as LearningExpectation[];
  return expectations;
}


  // -------- UPDATE LEARNING EXPECTATION SUBMISSION --------
  async updateLearningExpectation(
  courseId: string,
  submissionId: string,
  userId: string,
  answer: string
) {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );

    const envCheck = evaluateEnvironmentForUser(userId);
    const userData = await dataService.getUser(uid);
    const courseData = await dataService.getCourse(courseId);

    const submissionObj = {
      nameOfStudent:  userData?.name || "Unknown Student",
      nameOfSubmission: courseData?.name || "Unknown Submission",
      status: "Submitted",
      submittedAt: new Date().toISOString(),
      fileUrl: answer, 
      // Index 0 now represents abnormal login environment (location/device/time)
      cheatingStatus: [envCheck.isAbnormal, false, false],
      score: null,
      evaluation: null,
      miniQuizStatus: false,
      miniQuizQuestion: null,
      miniQuizAnswer: null,
    };

    await setDoc(ref, submissionObj, { merge: true });

    console.log("Updated successfully");
    return true;

  } catch (error) {
    console.error("Error update learning expectation:", error);
    return null;
  }
}

// -------- UPDATE SUBMISSION --------
//to do: update cheating status
async updateSubmission(
  courseId: string,
  submissionId: string,
  userId: string,
  fileUrl?: string,
  plagiarismLabel: 0 | 1 = 0,
  score?: number,
  evaluation?: string
) {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${userId}`);
    }

    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );
    
if (typeof fileUrl === "string" && fileUrl.length > 0) {
  const userData = await dataService.getUser(uid);
  const courseData = await dataService.getCourse(courseId);

  const hasPlagiarismSignal = plagiarismLabel === 1;
  const cheatingStatus: boolean[] = [false, hasPlagiarismSignal, false];
  const status = hasPlagiarismSignal ? "Cheated" : "Submitted";

  const submissionObj = {
    nameOfStudent: userData?.name || "Unknown Student",
    nameOfSubmission: courseData?.name || "Unknown Submission",
    status,
    submittedAt: new Date().toISOString(),
    fileUrl: fileUrl,
    cheatingStatus,
    score: null,
    evaluation: null,
    miniQuizStatus: false,
    miniQuizQuestion: null,
    miniQuizAnswer: null,
  };

  await setDoc(ref, submissionObj, { merge: true });

} else {
  await updateDoc(ref, {
    score: score ?? null,
    evaluation: evaluation ?? null,
  });
}
    console.log("Updated successfully");
    return true;

  } catch (error) {
    console.error("Error updating submission:", error);
    return null;
  }
}
// -------- GET USERS BY COURSE --------
async getUsersByCourse(courseId: string): Promise<User[]> {
  try {
    const usersRef = collection(db, "users");
    const q = query(usersRef, where("courses", "array-contains", courseId));
    const snap = await getDocs(q);
    
    const users: User[] = snap.docs.map(docSnap => {
      const userData = docSnap.data() as Partial<User>;
      return {
        uid: docSnap.id,               
        userId: userData.userId || "", 
        name: userData.name || "",
        role: userData.role as "student" | "staff" | "admin",
        email: userData.email || "",
        courses: userData.courses || [],
      };
    }).filter(u => u.role === "student"); 
    
    console.log(`Found ${users.length} users in course ${courseId}:`, users);
    return users;
    
  } catch (error) {
    console.error("Error getting users by course:", error);
    return [];
  }
}


// -------- GET ALL SUBMISSIONS FOR AN ITEM --------
async getSubmissionsForLearningExpectation(
  courseId: string,
  submissionId: string
): Promise<Record<string, SubmissionStatus>> {
  try {
    const submissionsRef = collection(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users"
    );
    
    const snap = await getDocs(submissionsRef);
    const submissions: Record<string, SubmissionStatus> = {};
    
    snap.forEach((docSnap) => {
      submissions[docSnap.id] = docSnap.data() as SubmissionStatus;
    });
    
    return submissions;
  } catch (error) {
    console.error("Error fetching submissions:", error);
    return {};
  }
}
  // -------- UPDATE MINI QUIZ SUBMISSION --------
  async updateSubmissionMiniQuiz(
  courseId: string,
  submissionId: string,
  userId: string,
  question: string,
  answer: string
) {
  try {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${userId}`);
    }
    const ref = doc(
      db,
      "courses",
      courseId,
      "submissions",
      submissionId,
      "users",
      uid
    );

    const snap = await getDoc(ref);

    if (!snap.exists()) return null;

    await updateDoc(ref, {
      miniQuizQuestion: question,
      miniQuizAnswer: answer,
      miniQuizStatus: true,
    });

    return true;
  } catch (error) {
    console.error("Error update mini quiz:", error);
    return null;
  }
}


  // -------- TEACHER REGRADES MINI QUIZ --------
  async updateSubmissionMiniQuizRegrade(courseId: string, submissionId: string, userId: string) {
    try {
         const uid = await this.getUidByUserId(userId);
        if (!uid) {
          throw new Error(`No UID found for userId: ${userId}`);
        }
        const ref = doc(
          db,
          "courses",
          courseId,
          "submissions",
          submissionId,
          "users",
          uid
        );

        const snap = await getDoc(ref);

        if (!snap.exists()) return null;

        await updateDoc(ref, {
          status :"Regraded",
          miniQuizStatus: false,
          miniQuizQuestion:null,
          miniQuizAnswer :null
        });

        return true;
      } catch (error) {
        console.error("Error regard mini quiz:", error);
        return null;
      }
  }


   async addCourse(course: {
    code: string;
    name: string;
    staffId: string;
    studentUIDs: string[];
    description: string;
    startDate: string;
    endDate: string;
  }) {
    // ===== CHECK DATE =====
    const start = new Date(course.startDate).getTime();
    const end = new Date(course.endDate).getTime();
    const now = Date.now();

    if (isNaN(start) || isNaN(end)) {
      throw new Error("Invalid date format");
    }

    if (start >= end) {
      throw new Error("Start date must be before end date");
    }

    if (end <= now) {
      throw new Error("End date must be in the future");
    }

    // ===== CHECK COURSE CODE EXISTS =====
    const courseQuery = query(
      collection(db, "courses"),
      where("code", "==", course.code)
    );

    const courseSnap = await getDocs(courseQuery);

    if (!courseSnap.empty) {
      throw new Error("Course code already exists");
    }

    // ===== CHECK STAFF EXISTS & ROLE =====
    const uid = await this.getUidByUserId(course.staffId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${uid}`);
    }
    const staffRef = doc(db, "users", uid);
    const staffSnap = await getDoc(staffRef);

    if (!staffSnap.exists()) {
      throw new Error("Staff ID does not exist");
    }

    const staffData = staffSnap.data();

    if (staffData.role !== "staff") {
      throw new Error("User is not a staff");
    }

    // ===== CHECK STUDENT IDS =====
    if (!course.studentUIDs || course.studentUIDs.length === 0) {
      throw new Error("Student IDs are required");
    }

    const studentUIDs: string[] = [];

    for (const studentId of course.studentUIDs) {
      const uid = await this.getUidByUserId(studentId);
      if (!uid) {
        throw new Error(`No UID found for userId: ${studentId}`);
      }

      const studentRef = doc(db, "users", uid);
      const studentSnap = await getDoc(studentRef);

      if (!studentSnap.exists()) {
        throw new Error(`Student with UID ${uid} does not exist`);
      }

      const studentData = studentSnap.data() as User;

      if (studentData.role !== "student") {
        throw new Error(`User ${studentId} (UID: ${uid}) is not a student`);
      }

      studentUIDs.push(uid);
    }

    // ===== CREATE COURSE =====
    const docRef = await addDoc(collection(db, "courses"), {
      code: course.code,
      name: course.name,
      staffId: course.staffId,
      studentUIDs: studentUIDs, 
      description: course.description,
      startDate: course.startDate,
      endDate: course.endDate,
      assignments: {},
      exercises: {},
      learningExpectations: {},
      createdAt: new Date().toISOString(),
    });

    return docRef.id;
  }

    // -------- REMOVE COURSE FROM USER --------
  async removeCourseFromUser(userId: string, courseId: string) {
    const uid = await this.getUidByUserId(userId);
    if (!uid) {
      throw new Error(`No UID found for userId: ${userId}`);
    }
    const userRef = doc(db, "users", uid);
    const snap = await getDoc(userRef);

    if (!snap.exists()) return;

    const data = snap.data() as User;

    const updatedCourses = (data.courses || []).filter(
      (id) => id !== courseId
    );

    await updateDoc(userRef, {
      courses: updatedCourses,
    });
  }

  // ===== DELETE COURSE (FULL CLEAN) =====
  async deleteCourseAndCleanup(courseId: string) {
    const courseRef = doc(db, "courses", courseId);
    const courseSnap = await getDoc(courseRef);

    if (!courseSnap.exists()) {
      throw new Error("Course not found");
    }

    const course = courseSnap.data() as Course;

    const staffUid = await dataService.getUidByUserId(course.staffId);

    if (!staffUid) {
      throw new Error("Staff UID not found");
    }

    const uids = [
      staffUid,
      ...(course.studentUIDs || []),
    ];

    await Promise.all(
    uids.map((uid) =>
      updateDoc(doc(db, "users", uid), {
        courses: arrayRemove(courseId),
      })
      )
    );

    await deleteFolder(courseId);
    await deleteDoc(courseRef);
  }

  // -------- UPDATE COURSE --------
  async updateCourse(courseId: string, data: Partial<Course>) {
    try {
      const courseRef = doc(db, "courses", courseId);
      // Check if document exists first
      const courseSnap = await getDoc(courseRef);
      if (!courseSnap.exists()) {
        throw new Error(`Course with ID ${courseId} does not exist`);
      }
      // Use updateDoc to work better with security rules
      await setDoc(courseRef, data, { merge: true });
      return true;
    } catch (error) {
      console.error("Error updating course:", error);
      return false;
    }
  }
}

export const dataService = new DataService();
```

---

### Bug 2: `userId` luôn rỗng trong [StaffSubmissionDetail.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmissionDetail.tsx)
- **Mức độ:** 🟡 Medium
- **File:** [StaffSubmissionDetail.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmissionDetail.tsx#L35)
- **Mô tả:** Biến `userId` (dòng 35) được khởi tạo là `""` nhưng **không bao giờ được set giá trị**. Nó được dùng tại dòng 442 để tạo link "Train Model":
```tsx
<Link to={`/staff/${userId}/learningExpectationTraining/${courseId}/${submissionId}`}>
```
→ Link sẽ bị sai: `/staff//learningExpectationTraining/...` (thiếu userId)

**Cách sửa:** Thêm `setUserId(auth.currentUser?.uid || "")` trong useEffect.

---

### Bug 3: Gọi [getUser](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/services/DataService.ts#395-419) hai lần không cần thiết — [StaffSubmissionDetail.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmissionDetail.tsx)
- **Mức độ:** 🟢 Low (Performance)
- **File:** [StaffSubmissionDetail.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmissionDetail.tsx#L100-L114)
- **Mô tả:** Trong vòng lặp, [getUser(uid)](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/services/DataService.ts#395-419) được gọi ở dòng 102 (trong `courseStudents`) rồi lại gọi **lần nữa** ở dòng 113 (`userData`). Đây là request Firebase thừa.

---

### Bug 4: [handleMarkNowClick](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmissionDetail.tsx#242-261) gọi [getUser](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/services/DataService.ts#395-419) với sai tham số
- **Mức độ:** 🟡 Medium
- **File:** [StaffSubmissionDetail.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmissionDetail.tsx#L242-L260)
- **Mô tả:** `student.userId` ở đây đã là `userId` (có thể là mã sinh viên), nhưng hàm lại gọi [getUser(student.userId)](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/services/DataService.ts#395-419) rồi lấy `userData.userId` — nếu `student.userId` không phải Firebase UID thì sẽ không tìm thấy user.

---

### Bug 5: `setStudentUserId` gọi 2 lần — [StudentHistory.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/StudentHistory.tsx)
- **Mức độ:** 🟢 Low
- **File:** [StudentHistory.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/StudentHistory.tsx#L60-L61)
- **Mô tả:** Dòng 60-61 gọi `setStudentUserId(currentStudentId)` **hai lần liên tiếp**, code thừa.

---

### Bug 6: `maxCount` được tính nhưng không sử dụng — [StudentDashboard.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/StudentDashboard.tsx)
- **Mức độ:** 🟢 Low (Dead code)
- **File:** [StudentDashboard.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/StudentDashboard.tsx#L195)
- **Mô tả:** Biến `maxCount` được tính toán nhưng **không bao giờ được sử dụng** — dead code.

---

### Bug 7: Student "Check now" dùng `auth.currentUser?.uid` thay vì `userId`
- **Mức độ:** 🟡 Medium
- **File:** [StudentDashboard.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/StudentDashboard.tsx#L338-L358)
- **Mô tả:** Trong phần Upcoming assignments, nút "Check now" dùng `auth.currentUser?.uid` (Firebase UID) làm `userId` trong URL, nhưng trang `SubmissionUpload` dùng `userId` (mã sinh viên) để query submission. **Nếu UID ≠ userId**, submission sẽ không tìm thấy.

---

### Bug 8: `SubmissionUpload` không tìm Learning Expectations
- **Mức độ:** 🟡 Medium
- **File:** [SubmissionUpload.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/SubmissionUpload.tsx#L49-L59)
- **Mô tả:** Khi tìm submission item, code chỉ kiểm tra `assignmentList` và `exerciseList` nhưng **bỏ qua `learningExpectations`**. Nếu student truy cập trang upload cho Learning Expectations, `submission` sẽ là `null` → trang hiển thị "Loading..." vĩnh viễn.

---

### Bug 9: Logic status luôn là "Missing" — [StudentDashboard.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/StudentDashboard.tsx)
- **Mức độ:** 🟢 Low
- **File:** [StudentDashboard.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/student/StudentDashboard.tsx#L131-L142)
- **Mô tả:** Logic xác định status có nhánh redundant:
```tsx
status = now > dueTime ? "Missing" : "Missing";  // Dòng 138 & 141
```
Cả hai nhánh đều trả về `"Missing"`. Nên sử dụng logic phù hợp hơn (ví dụ `"Pending"` nếu chưa đến deadline).

---

### Bug 10: Add modal thiếu label cho trường datetime — [StaffSubmission.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmission.tsx)
- **Mức độ:** 🟢 Low (UX)
- **File:** [StaffSubmission.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmission.tsx#L293-L309)
- **Mô tả:** Hai trường `datetime-local` trong modal Add không có label, người dùng không biết trường nào là "Opened Time" và trường nào là "Due Time".

---

### Bug 11: [Statistics.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/Statistics.tsx) — "Opened Date" được tính sai
- **Mức độ:** 🟡 Medium
- **File:** [Statistics.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/Statistics.tsx#L370-L372)
- **Mô tả:** "Opened Date" được **hardcode tính ngược 7 ngày** từ Due Date thay vì dùng `openedTime` thực tế từ data:
```tsx
openedDate: it.dueTime
  ? new Date(new Date(it.dueTime).getTime() - 7 * 24 * 60 * 60 * 1000).toLocaleString()
  : "-",
```
→ Hiển thị sai ngày mở.

---

### Bug 12: [Statistics.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/Statistics.tsx) — không hiển thị separate error/loading states
- **Mức độ:** 🟢 Low (UX)
- **File:** [Statistics.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/Statistics.tsx)
- **Mô tả:** Không có loading spinner hoặc skeleton khi trang đang tải dữ liệu ban đầu. Người dùng thấy "No courses found" trong vài giây trước khi data xuất hiện.

---

### Bug 13: [AdminCourses.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/admin/AdminCourses.tsx) — date format inconsistency
- **Mức độ:** 🟢 Low
- **File:** [AdminCourses.tsx](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/admin/AdminCourses.tsx#L76-L77)
- **Mô tả:** Khi edit course, `startDate.split('T')[0]` giả định format ISO string, nhưng nếu startDate không phải ISO string (ví dụ `"2024-03-15"`), kết quả vẫn đúng. Tuy nhiên nếu format khác thì sẽ sai.

---

## 📊 Tổng kết Bug

| Mức độ | Số lượng | Mô tả |
|--------|----------|-------|
| 🔴 Critical | 1 | Build error (ĐÃ SỬA) |
| 🟡 Medium | 5 | Logic bugs ảnh hưởng chức năng |
| 🟢 Low | 7 | UX, dead code, performance |

## 🎯 Khuyến nghị ưu tiên sửa

1. **Bug 8** — `SubmissionUpload` không tìm Learning Expectations → Student không thể upload cho loại này
2. **Bug 2** — `userId` rỗng trong link Train Model → Staff không thể train model
3. **Bug 7** — UID vs userId mismatch trong Student Dashboard → Có thể không tìm thấy submission
4. **Bug 11** — Opened Date bị hardcode sai → Hiển thị thông tin sai cho staff
5. **Bug 4** — [handleMarkNowClick](file:///home/khang/Desktop/Academic-Misconduct/amapp/src/pages/staff/StaffSubmissionDetail.tsx#242-261) gọi sai tham số → Có thể không mark được bài

## 🖼 Screenshots Test

````carousel
![Login Mobile](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/login_page_mobile_1773556464178.png)
<!-- slide -->
![Staff Courses](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/staff_courses_page_1773556533596.png)
<!-- slide -->
![Student History](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/student_history_page_1773556651016.png)
<!-- slide -->
![Profile Dropdown](/home/khang/.gemini/antigravity/brain/db62e02a-b1dc-4283-b86e-d08d075f4aec/staff_profile_dropdown_1773556573119.png)
````
