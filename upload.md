// Route for posting spare parts
router.post(
"/service/spares",
authMiddleware(["admin"]),
upload.single("image"),
sparesController
);

import multer from "multer";
import path from "path";

// Set up multer storage
const storage = multer.diskStorage({
destination: (req, file, cb) => {
cb(null, "uploads/");
},
filename: (req, file, cb) => {
cb(null, `${Date.now()}-${file.originalname}`);
},
});

const fileFilter = (
req: Express.Request,
file: Express.Multer.File,
cb: multer.FileFilterCallback
) => {
const filetypes = /jpeg|jpg|png/;
const extname = filetypes.test(path.extname(file.originalname).toLowerCase());
const mimetype = filetypes.test(file.mimetype);

if (extname && mimetype) {
return cb(null, true);
} else {
cb(new Error("Only images are allowed"));
}
};

const upload = multer({
storage,
fileFilter,
limits: { fileSize: 1024 _ 1024 _ 5 },
});

export default upload;

export const sparesController = async (req: AuthRequest, res: Response) => {
try {
if (!req.user || req.user.role !== "admin") {
return res.status(403).json({ message: "Unauthorized" });
}

    const { name, description, price } = req.body;
    const image = req.file
      ? /* req.file.path   */ `/uploads/${req.file.filename}`
      : "";
    const spares = new SpareParts({ name, description, price, image });
    await spares.save();
    res.status(200).json({ message: "Spares created successfully", spares });

} catch (error) {
res.status(500).json({ message: "Error creating spares", error });
}
};

index.ts / server.ts

app.use("/uploads", express.static("uploads"));
